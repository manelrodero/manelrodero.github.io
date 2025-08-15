---
layout : post
blog-width: true
title: 'Instalar AdGuard Home y Unbound en Docker'
date: '2025-08-06 07:47:04'
last-updated: '2025-08-11 10:08:04'
published: true
tags:
- HomeLab
- Raspberry
- Docker
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2025-08-06_cover.png"
thumbnail-img: ""
---

Hace casi tres años que utilizo [Pi-hole](https://pi-hole.net/){:target="_blank"} como servidor DNS y bloqueador de anuncios en mi HomeLab.

Inicialmente instalé [Pi-hole mediante Docker en Raspberry Pi OS Lite](instalacion-de-pihole-en-docker) y, hace poco más de un año, instalé un par de instancias de [Pi-hole y Unbound en contenedores LXC de Proxmox](instalar-pihole-en-proxmox-lxc).

Ahora le toca el turno a [**AdGuard Home**](https://github.com/AdguardTeam/AdGuardHome){:target="_blank"}, un proyecto muy similar en cuanto a características y rendimiento:

| Característica | Pi-hole | AdGuard Home |
| --- | --- | --- |
| **Interfaz web** | Simple, funcional | Moderna, con modo oscuro |
| **Instalación** | Script único, fácil en Linux | Binario único, muy fácil en Docker |
| **Bloqueo de anuncios** | Muy eficaz | Igual de eficaz, con más filtros |
| **DNS cifrado (DoH/DoT)** | No nativo, requiere configuración | Soporte nativo |
| **Parental control** | No incluido | Incluido |
| **DHCP integrado** | Básico | Más completo |
| **Consumo de recursos** | Menor (menos RAM/CPU) | Ligero pero algo más exigente |
| **Configuración avanzada** | Más flexible | Más amigable para principiantes |

## AdGuard Home

AdGuard Home es un software de red que permite bloquear anuncios y rastreos en todos los dispositivos domésticos sin necesidad de software cliente.

Funciona como un servidor DNS que redirige los dominios de rastreo a un "agujero negro" (_sinkhole_), impidiendo así que los dispositivos se conecten a esos servidores.

Se basa en el software usado en los [servidores DNS públicos de AdGuard](https://adguard-dns.io/){:target="_blank"} con los que comparte gran parte del código.

Se puede [instalar AdGuard Home usando Docker](https://github.com/AdguardTeam/AdGuardHome/wiki/Docker){:target="_blank"} mediante su [imagen oficial en Docker Hub](https://hub.docker.com/r/adguard/adguardhome){:target="_blank"}.

En el fichero de configuración `compose.yaml` se definen los puertos que se quieren exponer en función de los servicios que proporcionará AdGuard Home:

* DNS
* DHCP
* DNS-over-HTTPS (DoH)
* DNS-over-TLS
* DNS-over-QUIC
* DNSCrypt

La configuración que voy a usar inicialmente es la siguiente:

```yaml
services:
  adguardhome:
    image: adguard/adguardhome:v0.107.64
    container_name: adguardhome
    hostname: adguard
    restart: unless-stopped
    ports:
      # Servidor DNS "plain"
      - "53:53/tcp"
      - "53:53/udp"

      # Wizard inicial de AdGuard Home
      - "3000:3000/tcp"

      # Panel de administración (HTTP)
      - "80:80/tcp"

      # Panel de administración (HTTPS) / Servidor DNS-over-HTTPS (DoH)
      - "443:443/tcp"
      - "443:443/udp"

      # Los siguientes puertos están expuestos internamente por la imagen de AdGuard Home
      # porque están declarados como EXPOSE en su Dockerfile (pero no estarán accesibles
      # desde fuera del host a menos que se publiquen en este fichero)
      # ss -tuln | grep -E '67|68|853|5443|6060'

      # Servidor DNS-over-TLS
      #- "853:853/tcp"

      # Servidor DNS-over-QUIC
      #- "784:784/udp"
      #- "853:853/udp"
      #- "8853:8853/udp"

      # Servidor DNSCrypt
      #- "5443:5443/tcp"
      #- "5443:5443/udp"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - ./adguard_work:/opt/adguardhome/work
      - ./adguard_conf:/opt/adguardhome/conf
    networks:
      - dnsnet

networks:
  dnsnet:
    driver: bridge
```

El contenedor Docker se puede poner en marcha mediante el comando habitual:

```bash
docker compose up -d
```

A continuación, se puede acceder a `http://<IP>:3000` para completar el **asistente inicial** de AdGuard Home:

* Cambiar el tema a **Dark** ;-)
* Pulsar el botón `Get Started`
  * Admin Web Interface
    * Listen interface: All interfaces
    * Port: 80
  * DNS Server
    * Listen interface: All interfaces
    * Port: 53
  * Static IP Address
* Pulsar el botón `Next`
  * Authentication
    * Username: **admin**
    * Password: **********
    * Confirm password: **********
* Pulsar el botón `Next`
  * Configure your devices
  * Router
* Pulsar el botón `Next`
* Pulsar el botón `Open Dashboard`

Si todo está bien configurado, debería aparecer el panel de administración de AdGuard Home: 

![AdGuard Home][1]

A partir de aquí, se pueden explorar los `Settings` donde hay gran cantidad de cosas a configurar.

## Unbound

[**Unbound**](https://en.wikipedia.org/wiki/Unbound_(DNS_server)){:target="_blank"} es un _resolver_ de DNS **validador**, **recursivo** y de **almacenamiento en caché** desarrollado por [NLnet Labs](https://www.nlnetlabs.nl/projects/unbound/about/){:target="_blank"}.

Las imágenes Docker de Unbound más populares son:

* [Kyle Harding (@klutchell)](https://github.com/klutchell/unbound-docker){:target="_blank"}, de Balena.io
* [Matthew Vance](https://github.com/MatthewVance/unbound-docker){:target="_blank"}

Las diferencias principales son las siguientes:

| Característica | MatthewVance/unbound | klutchell/unbound |
| --- | --- | --- |
| **Popularidad y comunidad** | Muy usada, especialmente con Pi-hole | Menos conocida, pero bien mantenida |
| **Configuración por defecto** | Usa Cloudflare como forwarder (no recursivo) | Recursivo por defecto, sin forwarders |
| **Facilidad de personalización** | Muy flexible, permite montar tu `unbound.conf `| Igual de flexible, con ejemplos claros |
| **Arquitectura multiarch** | Compatible con ARM, x86, etc. | Multiarch también, con soporte _distroless_ |
| **Seguridad** | Buenas prácticas activadas por defecto | Imagen distroless (más segura y ligera) |
| **Licencia** | MIT | BSD-3-Clause |
| **Extras** | Soporte para `a-records.conf` locales | Soporte para Redis cache DB opcional |

Como la idea es usar Unbound como **resolver recursivo**, `klutchell/unbound` es ideal porque ya viene preparada para ello sin necesidad de modificar la configuración inicial.

En cualquier caso, se utilizará un fichero `unbound.conf` optimizado para garantizar:

* **Privacidad**: sin logs, sin identificadores únicos, sin forwarders
* [**DNSSEC**](https://www.icann.org/resources/pages/dnssec-what-is-it-why-important-2019-03-05-en){:target="_blank"}: validación de firmas para garantizar autenticidad
* **Rendimiento**: caché eficiente, mínimo uso de recursos

El fichero de configuración que voy a utilizar después de múltiples pruebas para optimizarlo es el siguiente:

```plaintext
server:

  # =====================
  # REGISTRO DE ACTIVIDAD
  # =====================

  # No registrar consultas (timestamp, IP, nombre, tipo y clase)
  log-queries: no

  # No registrar respuestas (timestamp, IP, nombre, tipo, clase, código de retorno, tiempo de resolución, de caché, tamaño)
  log-replies: no

  # Deshabilitar estadísticas en los log cada 'x' segundos para cada hilo
  statistics-interval: 0

  # Usar un fichero de log en lugar de 'syslog' (es decir, equivale a use-syslog: no)
  logfile: "/var/log/unbound.log"

  # Verbosidad de los logs de Unbound
  # 0 = solo errores
  # 1 = información operativa
  # 2 = información operativa detallada, incluyendo información breve por consulta
  # 3 = información a nivel de consulta y salida por consulta
  # 4 = información a nivel de algoritmo
  # 5 = registra la identificación del cliente en caso de fallas de caché
  verbosity: 1

  # ==================
  # INTERFACE Y PUERTO
  # ==================

  # Imprescindible escuchar en todas las interfaces al usar 'unbound' en Docker
  # El fichero 'unbound.conf' de la imagen de @klutchell ya define la interfaz
  # https://github.com/klutchell/unbound-docker/blob/main/rootfs_overlay/etc/unbound/unbound.conf
  # interface: 0.0.0.0

  # Puerto de Unbound (en el 53 está AdGuard Home)
  port: 5335

  # ==========================
  # PROTOCOLOS (IPv4, no IPv6)
  # ==========================

  # Utilizar IPv4 únicamente para consultas/respuestas
  do-ip4: yes
  do-ip6: no

  # Utilizar tanto UDP como TCP
  do-udp: yes
  do-tcp: yes

  # Preferir el uso de IPv4, al no haber IPv6 nativo
  prefer-ip4: yes
  prefer-ip6: no

  # ======================
  # SEGURIDAD Y PRIVACIDAD
  # ======================

  # Rechazar las peticiones 'id.server' y 'hostname.bind'
  hide-identity: yes

  # Rechazar las peticiones 'version.server' y 'version.bind'
  hide-version: yes

  # Confiar en Glue solo si está dentro de la autoridad del servidor
  harden-glue: yes

  # Requerir datos DNSSEC para zonas ancladas en la confianza (trust-anchored),
  # si dichos datos no están presentes, la zona se vuelve falsa (Bogus)
  harden-dnssec-stripped: yes

  # Realizar consultas adicionales para validar los registros NS y
  # sus direcciones IP (glue records) en la cadena de delegación
  # Aplica DNSSEC a esos datos si están firmados
  # Protege contra ataques como NS hijacking o manipulación de delegaciones
  # Más estricto que la validación DNSSEC estándar y no está en el RFC
  harden-referral-path: yes

  # Niveles de profundidad a seguir para validar los targets (NS y glue records)
  # 3 -> registro A (dirección IPv4 del servidor NS)
  # 2 -> registro AAAA (dirección IPv6 del servidor NS)
  # 1 -> registro MX (servidores de correo)
  # 0 -> registro SRV (servicios específicos: SIP, LDAP, etc.)
  # 0 -> registro PTR (resolución inversa IP -> nombre)
  target-fetch-policy: "3 2 1 0 0"

  # Utilizar bits aleatorios codificados en 0x20 en la consulta para frustrar intentos de suplantación
  # Pi-hole recomendaba no usar 'caps-for-id' a causa de problemas en DNSSEC:
  # + https://discourse.pi-hole.net/t/unbound-stubby-or-dnscrypt-proxy/9378
  # Como no hay errores en los logs, se deja en "yes" para mejorar la protección contra spoofing
  use-caps-for-id: yes

  # ======
  # DNSSEC
  # ======

  # DNSSEC (Domain Name System Security Extensions)
  # https://www.icann.org/resources/pages/dnssec-what-is-it-why-important-2019-03-05-en

  # Se configura 'unbound' para realizar la validación criptográfica de DNSSEC
  # Para ello se usa la 'root trust anchor' (ancla de confianza raíz) definida en el fichero 'root.key'
  # + https://datatracker.ietf.org/doc/html/rfc5011.html
  # El Dockerfile de la imagen de @klutchell se encarga de obtener el fichero 'root.key'
  # https://github.com/klutchell/unbound-docker/blob/main/Dockerfile
  # auto-trust-anchor-file: root.key

  # Enviar una consulta de etiqueta clave RFC 8145 después de preparar el ancla de confianza
  # + https://datatracker.ietf.org/doc/html/rfc8145.html
  trust-anchor-signaling: yes

  # El validador registra los fallos de validación, independientemente del nivel de verbosity
  val-log-level: 2

  # ===================
  # RENDIMIENTO Y CACHÉ
  # ===================

  # En Pi-hole usaba 1 hilo y, aunque debería ser suficiente, según Copilot se podría aumentar:
  # + Red doméstica (hasta ~20 dispositivos) -> 1
  # + Red media (~20–100 dispositivos) -> 2
  # + Red alta carga o tráfico DNS intenso -> 4
  # Según la documentación "Performance Tuning" de Unbound:
  # Debería ser igual al número de núcleos del sistema (Raspberry Pi 4):
  # - CPU: Broadcom BCM2711
  # - Arquitectura: ARM Cortex-A72 (ARMv8-A de 64 bits)
  # - Número de núcleos (cores): 4 núcleos
  # Para ver el número de hilos en uso, se puede ejecutar:
  # docker exec -it unbound unbound-control status | grep -i num-threads
  num-threads: 4

  # Según la documentación "Performance Tuning" de Unbound:
  # Asegurar búfer del kernel suficientemente grande para no perder mensajes en picos de tráfico
  so-rcvbuf: 4m
  so-sndbuf: 4m
  msg-cache-size: 50m
  rrset-cache-size: 100m

  # Tiempo de vida máximo (1 día) y mínimo (1 hora) para RRsets y mensajes en la caché
  cache-max-ttl: 86400
  cache-min-ttl: 3600

  # Tiempo de vida máximo (1 hora) y mínimo (5 minutos) para respuestas negativas,
  # estas tienen una SOA en la sección de autoridad que es limitada en el tiempo
  cache-max-negative-ttl: 3600
  cache-min-negative-ttl: 300

  # ==========
  # ROOT HINTS
  # ==========

  # Lista de servidores raíz del DNS (los 13 servidores raíz como a.root-servers.net, etc.), con sus direcciones IP
  # El Dockerfile de la imagen Unbound de @klutchell se encarga de obtener el fichero 'root.hints'
  # https://github.com/klutchell/unbound-docker/blob/main/Dockerfile
  # root-hints: root.hints

  # ==============================
  # CONTROL DE ACCESO Y PRIVACIDAD
  # ==============================

  # Permitir consultas a este servidor que se originen desde las siguientes direciones IP
  access-control: 127.0.0.1 allow
  access-control: 192.168.0.0/16 allow

  # Direcciones de la red privada que no se pueden devolver para nombres de internet públicos
  # https://datatracker.ietf.org/doc/html/rfc1918.html
  private-address: 192.168.0.0/16
  private-address: 169.254.0.0/16
  private-address: 172.16.0.0/12
  private-address: 10.0.0.0/8
  private-address: fd00::/8
  private-address: fe80::/10

  # Permitir que este dominio y todos sus subdominios contengan direcciones privadas
  private-domain: yourdomain.com

  # Enviar una cantidad mínima de información a los servidores upstream para mejorar la privacidad
  qname-minimisation: yes

  # QNAME estricto: no se envía el QNAME completo a servidores DNS potencialmente dañados
  # Muchos dominios no se podrán resolver cuando esta opción esté habilitada (p.ej. HBO)
  # qname-minimisation-strict: yes

  # Usar la cadena DNSSEC NSEC para sintetizar NXDOMAIN y otras denegaciones,
  # utilizando información de respuestas NXDOMAIN anteriores
  aggressive-nsec: yes

  # ======================
  # EDNS BUFFER Y PREFETCH
  # ======================

  # Número de bytes que se anunciarán como tamaño del búfer de reensamblaje de EDNS
  # Búfer recomendado para evitar problemas de fragmentación al usar UDP en las consultas/respuestas
  # Este valor se basa en un estudio de NLnet Labs y el DNS Flag Day 2020
  # https://dnsflagday.net/2020/
  edns-buffer-size: 1232

  # Precarga de entradas de caché próximas a caducar para dominios consultados frecuentemente
  # Genera aproximadamente un 10 por ciento más de tráfico y carga en la máquina,
  # pero las entradas populares no caducan de la caché
  prefetch: yes

  # Configuración "validator iterator" para activar la validación DNSSEC
  # También hay que configurar anclas de confianza (trust-anchors) para que la validación sea útil
  # No se usa el módulo 'subnetcache' para evitar errores como el siguiente:
  # warning: subnetcache: prefetch is set but not working for data originating from the subnet module cache
  module-config: "validator iterator"

# ==============
# CONTROL REMOTO
# ==============

remote-control:
  control-enable: yes
  # La interfaz de control remoto por defecto escucha en 127.0.0.1 y ::1 (puerto 8953)
  # Se utiliza un socket para no evitar el uso de certificados
  control-interface: /run/unbound.ctl
```

Para utilizar esta configuración, se necesitan un par de ficheros adicionales:

* `root.key`: fichero generado por la IANA que contiene la **clave pública** de la **zona raíz del DNS**. Este fichero, firmado por la **clave raíz de DNSSEC**, contiene la clave que permite validar las respuestas DNS mediante este protocolo
* `root.hints`: fichero que contiene una **lista de servidores raíz del DNS**. Son los primeros puntos de contacto que Unbound usa para empezar a resolver cualquier nombre de dominio

> **Nota**: Al usar la imagen de **@klutchell** no es necesario ejecutar el siguiente comando `docker run` para obtener por nuestra cuenta estos ficheros ya que la propia imagen se encarga de hacerlo.

Para obtener el fichero `root.key` se necesita usar el comando `unbound-anchor` instalado en el sistema o desde el mismo contenedor de Unbound.

El fichero `root.hints`, también llamado `named.root`, se puede descargar desde [InterNIC](https://www.internic.net/domain/named.root).

El siguiente comando se encarga de hacerlo usando una pequeña imagen de Alpine:

```bash
docker run --rm \
  -v "$(pwd)/unbound:/unbound" \
  alpine \
  sh -c "apk add --no-cache curl unbound && \
         curl -sSL https://www.internic.net/domain/named.root -o /unbound/root.hints && \
         unbound-anchor -a /unbound/root.key"
```

Una vez tenemos los ficheros necesarios para ejecutar Unbound, se puede añadir el siguiente código al fichero `compose.yaml`:

```yaml
  unbound:
    image: klutchell/unbound:v1.23.1
    container_name: unbound
    restart: unless-stopped
    ports:
      - "5335:5335/tcp"
      - "5335:5335/udp"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - ./unbound/unbound.conf:/etc/unbound/custom.conf.d/unbound.conf

      # Si se quiere log externo, se crea este fichero y se define en 'unbound.conf'
      - ./unbound/unbound.log:/var/log/unbound.log

      # El Dockerfile de esta imagen descarga el fichero 'root.hints' y genera el fichero 'root.key'
      # https://github.com/klutchell/unbound-docker/blob/main/Dockerfile
      #- ./unbound/root.hints:/var/lib/unbound/root.hints
      #- ./unbound/root.key:/var/lib/unbound/root.key

      # Directorio para el socket 'unbound.ctl' (remote control)
      - ./unbound/run:/run
    networks:
      - dnsnet
```

Y poner en marcha el contenedor de la forma habitual:

```bash
docker compose up -d
```

## AdGuard Home + Unbound

Ahora que ya están configurados tanto AdGuard Home como Unbound, se pueden unir para que el **upstream server** sea Unbound y no `https://dns10.quad9.net/dns-query`:

* Se accede al panel de administración de AdGuard Home
* Se accede a **Settings** &rarr; **DNS Settings**
* Se cambia el los _Upstream DNS servers_ por los siguientes:

```plaintext
udp://<IP>:5335
tcp://<IP>:5335
```

> **Nota**: `<IP>` es la dirección IP del _host_ (en mi caso la Raspberry Pi)

Una vez hecho ésto, se puede pulsar el botón `Test upstreams` y, si todo funciona correctamente, pulsar el botón `Apply`.

## Optimización y _troubleshooting_

### `so-rcvbuf` y `so-sndbuf`

Cuando se configuró un búfer de recepción `so-rcvbuf` y uno de envío `so-sndbuf` de `1m`:

```yaml
  so-rcvbuf: 1m
  so-sndbuf: 1m
```

En los registros de Unbound se podían observar advertencias similares a las siguientes porque el _host_ no le proporcionaba ese valor:

```plaintext
unbound      | [1754646979] unbound[1:0] warning: so-rcvbuf 1048576 was not granted. Got 425984. To fix: start with root permissions(linux) or sysctl bigger net.core.rmem_max(linux) or kern.ipc.maxsockbuf(bsd) values.
unbound      | [1754646979] unbound[1:0] warning: so-sndbuf 1048576 was not granted. Got 425984. To fix: start with root permissions(linux) or sysctl bigger net.core.wmem_max(linux) or kern.ipc.maxsockbuf(bsd) values.
```

En la máquina host, en mi caso la Raspberry Pi, se puede conocer los valores por defecto usando los siguientes comandos:

```bash
sysctl net.core.rmem_max
sysctl net.core.wmem_max
```

Para cambiar estos valores de forma persistente, es necesario modificar el fichero `/etc/sysctl.conf` de la siguiente manera:

```bash
echo "net.core.rmem_max=1048576" | sudo tee -a /etc/sysctl.conf
echo "net.core.wmem_max=1048576" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### Verificación de DNSSEC

Para probar el funcionamiento del servidor DNS de AdGuard Home y de Unbound, lo mejor es utilizar el comando `dig` que forma parte del paquete `dnsutils` (este paquete también incluye los comandos `nslookup` y `host`).

Se puede instalar este paquete en la Raspberry Pi usando `apt` de la manera habitual:

```bash
sudo apt install dnsutils
```

Una vez instalado, se puede comprobar que **DNSSEC** está funcionando correctamente consultando `dnssec-tools.org` y viendo que obtenemos un resultado **NOERROR** como el mostrado a continuación:

```bash
dig @127.0.0.1 -p 5335 dnssec-tools.org
```

```plaintext
; <<>> DiG 9.18.33-1~deb12u2-Debian <<>> @127.0.0.1 -p 5335 dnssec-tools.org
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 45454
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;dnssec-tools.org.              IN      A

;; ANSWER SECTION:
dnssec-tools.org.       3600    IN      A       185.199.109.153
dnssec-tools.org.       3600    IN      A       185.199.110.153
dnssec-tools.org.       3600    IN      A       185.199.111.153
dnssec-tools.org.       3600    IN      A       185.199.108.153

;; Query time: 319 msec
;; SERVER: 127.0.0.1#5335(127.0.0.1) (UDP)
;; WHEN: Fri Aug 08 12:50:39 CEST 2025
;; MSG SIZE  rcvd: 109
```

Para verificar DNSSEC en un dominio incorrectamente configurado, se puede usar `dnssec-failed.org` y el resultado debería ser **SERVFAIL** tal como se muestra a continuación:

```bash
dig @127.0.0.1 -p 5335 dnssec-failed.org
```

```plaintext
; <<>> DiG 9.18.33-1~deb12u2-Debian <<>> @127.0.0.1 -p 5335 dnssec-failed.org
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 7606
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;dnssec-failed.org.             IN      A

;; Query time: 691 msec
;; SERVER: 127.0.0.1#5335(127.0.0.1) (UDP)
;; WHEN: Fri Aug 08 12:51:14 CEST 2025
;; MSG SIZE  rcvd: 46

```

### `unbound-control`

El comando `unbound-control` permite interactuar con Unbound en tiempo real, sin tener que reiniciar el servicio ni modificar el fichero de configuración.

Algunas de sus funciones más útiles son:

* Recargar la configuración sin reiniciar: `unbound-control reload`
* Vaciar la caché DNS: `unbound-control flush` o `unbound-control flush www.youtube.com`
* Consultar el estado del servidor: `unbound-control status`
* Ver estadísticas en tiempo real: `unbound-control stats`
* Buscar en la caché: `unbound-control lookup www.youtube.com`

En mi caso, al usar Unbound en un contenedor Docker, la forma más fácil de ejecutar el comando es usar `docker exec` de la siguiente manera:

```bash
docker exec unbound unbound-control status
```

```plaintext
version: 1.23.1
verbosity: 3
threads: 4
modules: 2 [ validator iterator ]
uptime: 363 seconds
options: reuseport control(namedpipe)
unbound (pid 1) is running...
```

### `unbound-checkconf`

El comando `unbound-checkconf` permite comprobar que la configuración de Unbound sea correcta:

```bash
docker exec unbound unbound-checkconf
```

Además, también se puede utilizar para comprobar cuál es la **configuración efectiva** de un determinado parámetro (útil cuando se utilizan ficheros de configuración _custom_ que sobreescriben la configuración por defecto de la imagen):

```bash
docker exec -it unbound unbound-checkconf -o num-threads
```

```plaintext
4
```

### Monitorizar el rendimiento

Se puede usar el comando `docker stats` para ver el consumo de recursos de cada contenedor Docker:

```plaintext
CONTAINER ID   NAME            CPU %     MEM USAGE / LIMIT   MEM %     NET I/O           BLOCK I/O    PIDS
635720702d8a   adguardhome     0.03%     0B / 0B             0.00%     2.14MB / 2.28MB   0B / 0B      12
e79b95133016   unbound         0.00%     0B / 0B             0.00%     4.72MB / 1.73MB   0B / 0B      2
```

Tambíen se puede usar el comando `htop` para ver el uso de CPU por cada núcleo de los procesos de Unbound y AdGuard Home:

```bash
htop -p $(pgrep -d',' -f 'unbound|adguardhome')
```

![htop][2]

### Ficheros de Unbound

El contenido del directorio `./unbound` que utiliza el contenedor Docker es el siguiente:

```plaintext
unbound/:
total 6252
-rw-r----- 1 _rpc input    3310 Aug  8 12:19 root.hints
-rw-r----- 1 _rpc input    1250 Aug  8 13:02 root.key
drwxr-xr-x 2 _rpc input    4096 Aug 11 11:22 run
-rw-r----- 1 _rpc input    5651 Aug 11 10:50 unbound.conf
-rw-r----- 1 _rpc input 6378914 Aug 11 11:33 unbound.log

unbound/run:
total 0
srw-rw---- 1 _rpc input 0 Aug 11 11:22 unbound.ctl
```

Se puede observar que el propietario de los ficheros es `_rpc:input` (equivale a `101:102`) tal como requiere la imagen de Unbound de @klutchell.

### [Performance Tuning](https://unbound.docs.nlnetlabs.nl/en/latest/topics/core/performance.html){:target="_blank"}

Se han cambiado algunos valores de la configuración según el documento "[Performance Tuning](https://unbound.docs.nlnetlabs.nl/en/latest/topics/core/performance.html){:target="_blank"} de la documentación de Unbound:

* `num-threads`: igual al número de núcleos de la CPU del sistema (**4** en el caso de una Raspberry Pi)
* `so-rcvbuf` y `so-sndbuf`: `4m` (**4194304**) para servidores con mucha demanda

Algunos valores que no se han cambiado por tener valores correctos por defecto son los siguientes:

* `so-reuseport: yes`, UDP más rápido con multiproceso (sólo en Linux)
* `msg-cache-slabs: 4`, potencia de 2 cercana al `num-threads`
  * `rrset-cache-slabs: 4`
  * `infra-cache-slabs: 4`
  * `key-cache-slabs: 4`
* `rrset-cache-size: 100m`, más memoria caché, rrset=msg*2
  * `msg-cache-size: 50m`
* `outgoing-range: 8192`, la imagen de @klutchell usa `libevent` y las conexiones salientes no están limitadas a 1024
  * `num-queries-per-thread: 4096`

## _Tuning_ de AdGuard Home

Algunos cambios adicionales en AdGuard Home son los siguientes:

* Settings
  * DNS settings
    * DNS server configuration
      * Rate limit: 20 &rarr; **50**
* Filters
  * DNS blocklists
    * Add blocklist > Choose from the list
      * **OISD Blocklist Big**
      * **Steven Black's List**
    * Activar `AdAway Default Blocklist`

## Cambiar contraseña de AdGuard Home

La contraseña del panel de administración de AdGuard Home se guarda en formato **hash bcrypt** en el fichero `AdGuardHome.yaml`.

Se puede generar una contraseña en este formato usando un contenedor Docker temporal de [`httpd`](https://hub.docker.com/_/httpd){:target="_blank"}:

```bash
docker run --rm httpd:2.4 htpasswd -B -n -b admin LaContraseñaDeAdGuardHome
```

En pantalla se verá algo así:

```plaintext
admin:$2y$05$32Fn.bwDVpMYHZ5L6/Ql/OSyDUEz19ZiQ.CRSICkFfBHs8mKFr3WO
```

En el fichero `AdGuardHome.yaml` se modificará la siguiente línea con el _hash_ anterior:

```yaml
users:
  - name: admin
    password: $2y$05$32Fn.bwDVpMYHZ5L6/Ql/OSyDUEz19ZiQ.CRSICkFfBHs8mKFr3WO
```

A continuación, se reinicia el contenedor de AdGuard Home.

## Referencias

* [Pi-hole vs AdGuard Home: Network-Wide Ad Blocking Solution Performance Test 2025](https://markaicode.com/pihole-vs-adguard-home-performance-test/){:target="_blank"}, MARK Ai code, 27 May 2025
* [Learn How to Run AdGuard Home using Docker](https://pimylifeup.com/adguard-home-docker/){:target="_blank"}, PiMyLifeUp, 21 Jul 2024
* [How to use Unbound with AdGuard Home or Pi-hole](https://dev.to/lithium_dreams_echo/how-to-use-unbound-with-adguard-home-1o5n){:target="_blank"}, Dev.To, 30 Jan 2024
* [Home Assistant Community Add-on: AdGuard Home](https://github.com/hassio-addons/addon-adguard-home){:target="_blank"}, Franck Nijhof
* [AdGuardHome sync](https://github.com/bakito/adguardhome-sync){:target="_blank"}, Marc Brugger
* [Installing Unbound for Pi-Hole on the Raspberry Pi](https://pimylifeup.com/raspberry-pi-unbound/){:target="_blank"}, PiMyLifeUp, 10 Oct 2023
* [HomeLab: AdGuard: The One Reason to Use Unbound DNS ( IMHO )](https://medium.com/@life-is-short-so-enjoy-it/homelab-adguard-the-one-reason-to-use-unbound-dns-imho-3ee297377365){:target="_blank"}, 23 Oct 2023
* [Unbound documentation](https://unbound.docs.nlnetlabs.nl/en/latest/index.html){:target="_blank"}
  * [unbound.conf(5)](https://unbound.docs.nlnetlabs.nl/en/latest/manpages/unbound.conf.html){:target="_blank"}
  * [Example of unbound.conf file](https://github.com/NLnetLabs/unbound/blob/master/doc/example.conf.in){:target="_blank"}

### Historial de cambios

* **2025-08-06**: Documento inicial
* **2025-08-08**: Configuración optimizada de Unbound/AdGuard
* **2025-08-11**: Cambios en 'unbound.conf'

[1]: /assets/img/blog/2025-08-06_image_1.png "AdGuard Home"
[2]: /assets/img/blog/2025-08-06_image_2.png "htop"
