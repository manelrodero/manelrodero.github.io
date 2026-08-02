---
layout : post
blog-width: true
title: 'DNSSEC, NTP y el problema del huevo y la gallina en una Raspberry Pi sin RTC'
date: '2026-08-02 19:36:38'
#last-updated: '2026-08-02 19:36:38'
published: true
tags:
- Raspberry
author:
  display_name: Manel Rodero
#cover-img: "/assets/img/blog/2026-08-02_cover.png"
thumbnail-img: ""
---

*Cómo un reinicio tras varios días apagada dejaba sin Internet a toda la red, y por qué la solución no tenía nada que ver con el DNS en sí.*

## El síntoma

Una Raspberry Pi 4 con Docker corriendo [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome) + [Unbound](https://github.com/klutchell/unbound-docker) como servidor DNS local de la red doméstica (`192.168.1.50`), con un segundo servidor idéntico como respaldo en otra máquina (`192.168.1.80`, normalmente apagado para pruebas).

Tras un reinicio de la Raspberry Pi -especialmente después de haber estado apagada varios días (por ejemplo, durante unas vacaciones)- toda la red se quedaba sin resolución DNS. Cualquier PC de la red, incluido uno con Windows configurado para usar ambos servidores como DNS, mostraba:

```
C:\Users\Manel>nslookup google.com
Server:  UnKnown
Address:  192.168.1.50
*** UnKnown can't find google.com: Server failed
```

Lo más desconcertante: si en ese momento se editaba a mano `/etc/resolv.conf` en la Raspberry Pi y se añadía temporalmente un tercer DNS externo (por ejemplo `1.1.1.1`), al cabo de unos segundos todo volvía a funcionar. Y lo seguía haciendo aunque después se retirara ese DNS temporal y se dejaran solo los dos servidores originales. El problema no volvía a aparecer... hasta el próximo reinicio tras una parada larga.

## La teoría: un círculo cerrado entre DNS y reloj

La Raspberry Pi no tiene reloj de hardware (RTC). Al arrancar, no tiene forma nativa de saber qué hora es hasta que consigue sincronizarla por red mediante NTP (`systemd-timesyncd`), y para eso necesita resolver por DNS el nombre del servidor NTP.

Por otro lado, Unbound tenía activada la validación DNSSEC (`harden-dnssec-stripped: yes`), que exige que la hora del sistema sea correcta: las firmas criptográficas de las zonas DNS (registros RRSIG) solo son válidas dentro de una ventana temporal concreta. Si el reloj está desfasado lo suficiente, Unbound rechaza las respuestas como `bogus` y devuelve `SERVFAIL`.

El resultado es un bucle cerrado:

1. Para tener DNS necesitas que Unbound valide correctamente las firmas DNSSEC.
2. Para validar DNSSEC necesitas tener la hora correcta.
3. Para tener la hora correcta necesitas sincronizar por NTP.
4. Para sincronizar por NTP necesitas resolver un nombre de dominio por DNS.
5. Vuelta al punto 1.

Mientras el desfase del reloj al arrancar sea pequeño (minutos, quizás horas), las ventanas de validez de las firmas DNSSEC lo absorben sin problema y todo funciona con normalidad. El problema solo se manifiesta cuando la Raspberry Pi lleva apagada el tiempo suficiente como para que ese desfase supere la ventana de validez de las firmas.

## Diagnóstico paso a paso

### 1. Confirmar el estado del reloj al fallar

```bash
timedatectl status
```

```
System clock synchronized: no
              NTP service: active
```

### 2. Confirmar el error exacto en Unbound

```
info: validation failure <google.com. A IN>: SERVFAIL [exceeded the maximum number of sends] no DS for DS google.com. while building chain of trust
```

Este mensaje es la firma característica de un fallo de validación DNSSEC, no de un problema de conectividad o de configuración del propio Unbound.

### 3. Descubrir por qué a veces "se cura solo"

Aquí apareció la parte más interesante: tras apagados cortos (minutos), el problema no se reproducía, aunque el reloj tampoco estuviera sincronizado por NTP todavía. La explicación está en **dos mecanismos independientes que "recuerdan" la última hora conocida entre reinicios**, sin que sea NTP el que la restaura:

- **`fake-hwclock`**: simula un reloj de hardware guardando periódicamente la hora del sistema en `/etc/fake-hwclock.data`, y la restaura al arrancar antes de que exista red.
- **`systemd-timesyncd`**: guarda también, de forma independiente, la fecha de la última sincronización con éxito como el `mtime` del fichero `/var/lib/systemd/timesync/clock`. Al arrancar, si el reloj del sistema va por detrás de ese `mtime`, systemd lo adelanta directamente a esa fecha, **antes de contactar con ningún servidor NTP**.

Ambos ficheros se actualizan periódicamente mientras el sistema está encendido y sincronizado. Por eso, tras un apagado corto, la hora "recordada" seguía siendo lo bastante reciente como para pasar la validación DNSSEC sin problema - sin que hiciera falta que el NTP real llegara a completarse. El sistema arrancaba con una hora "aproximadamente correcta" heredada de la sesión anterior, y esa aproximación era suficiente.

El problema solo aparecía cuando el sistema llevaba apagado tanto tiempo (varios días) que esa hora heredada quedaba demasiado desfasada respecto a la real.

## Las pruebas realizadas

Para confirmar la teoría sin tener que esperar días de forma pasiva, se reprodujo el fallo de forma controlada, envejeciendo a mano **los dos mecanismos de persistencia de hora**, no solo uno:

```bash
sudo systemctl stop systemd-timesyncd
sudo date -s "2026-07-20 00:00:00"
sudo fake-hwclock save
sudo touch -d "2026-07-20 00:00:00" /var/lib/systemd/timesync/clock
sudo shutdown -h now
```

Un primer intento envejeciendo solo `fake-hwclock` (sin tocar `/var/lib/systemd/timesync/clock`) **no reprodujo el fallo**: el `mtime` de ese segundo fichero seguía siendo reciente (se había actualizado apenas minutos antes gracias a que el DNS ya funcionaba), y bastó para que systemd adelantara el reloj a una hora suficientemente correcta antes de que se disparara ningún problema de DNSSEC. Esto en sí mismo fue una confirmación valiosa: había **dos fuentes de persistencia de hora**, no una, y ambas debían envejecerse para forzar el escenario real.

Con ambos ficheros envejecidos ~13 días y tras un apagado real (incluyendo desconexión de corriente), el arranque mostró:

```
timedatectl status
System clock synchronized: no
```

```
nslookup onedrive.com
;; Got SERVFAIL reply from 192.168.1.50, trying next server
;; communications error to 192.168.1.80#53: timed out
;; no servers could be reached
```

Fallo reproducido de forma fiable y controlada, sin depender de dejar el equipo apagado varios días de verdad.

## La solución: eliminar la dependencia circular

El error de diseño no estaba en Unbound ni en AdGuard Home, sino en que la sincronización horaria dependía de una resolución DNS previa. La solución consiste en romper esa dependencia usando **direcciones IP fijas** como servidores NTP, en lugar del valor por defecto de Debian (`pool.debian.org`), que es un nombre DNS de tipo *round-robin*:

```bash
timedatectl show-timesync --all | grep -i ntp
```

```
FallbackNTPServers=0.debian.pool.ntp.org 1.debian.pool.ntp.org 2.debian.pool.ntp.org 3.debian.pool.ntp.org
```

Se edita `/etc/systemd/timesyncd.conf`:

```ini
[Time]
NTP=162.159.200.1 216.239.35.0 216.239.35.4
```

Estas direcciones no son arbitrarias:

- `162.159.200.1` → Cloudflare (`time.cloudflare.com`), servida en *anycast* desde decenas de datacenters, siempre responde el nodo más cercano y la IP nunca cambia.
- `216.239.35.0` / `216.239.35.4` → Google (`time.google.com`), mismo principio de *anycast*.

A diferencia de los servidores individuales del pool NTP (que pueden desaparecer o cambiar), estas IPs de proveedores grandes ofrecen estabilidad a largo plazo.

Aplicar y reiniciar:

```bash
sudo systemctl restart systemd-timesyncd
sudo reboot
```

## Verificación final

Con el fix aplicado, se repitió la prueba de envejecimiento de los dos ficheros de hora (~13 días de desfase) y reinicio completo. Resultado:

```
timedatectl status
System clock synchronized: yes
```

Tras unos segundos (el tiempo que tarda Docker en levantar los contenedores), la resolución DNS funcionó con normalidad, sin necesidad de tocar `/etc/resolv.conf` ni de ninguna intervención manual:

```
nslookup onedrive.com
Server:         192.168.1.50
Address:        192.168.1.50#53
Non-authoritative answer:
Name:   onedrive.com
Address: 20.101.246.164
```

## Conclusiones

- Un servidor DNS local con validación DNSSEC estricta introduce una dependencia oculta con la hora del sistema.
- En equipos sin reloj de hardware (como la Raspberry Pi), esa dependencia se convierte en un problema real tras apagados largos, porque la sincronización NTP por defecto depende a su vez de una resolución DNS previa.
- Mecanismos como `fake-hwclock` y el propio `systemd-timesyncd` "disimulan" el problema en apagados cortos al recordar la última hora conocida, lo que hace que el fallo parezca intermitente y difícil de reproducir.
- La solución no toca ni AdGuard Home ni Unbound: basta con apuntar `systemd-timesyncd` a IPs NTP fijas y estables (anycast de Cloudflare o Google) para eliminar la dependencia circular por completo.
- Como medida adicional de resiliencia, conviene mantener activo el segundo servidor DNS de respaldo (`192.168.1.80`) para no depender de un único punto de fallo en la red.

### Historial de cambios

* **2026-08-02**: Documento inicial
