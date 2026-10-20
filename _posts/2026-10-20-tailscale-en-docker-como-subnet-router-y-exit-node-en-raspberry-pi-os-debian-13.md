---
layout : post
blog-width: true
title: 'Tailscale en Docker como Subnet Router y Exit Node en Raspberry Pi OS (Debian 13)'
date: '2026-10-20 20:12:41'
#last-updated: '2026-10-20 20:12:41'
published: true
tags:
- Raspberry
author:
  display_name: Manel Rodero & Claude
cover-img: "/assets/img/blog/2026-10-20_cover.png"
thumbnail-img: ""
---

Tras actualizar mi Raspberry Pi 4 a **Raspberry Pi OS Lite basado en Debian 13 (Trixie)**, el contenedor Docker de [Tailscale](https://tailscale.com/){:target="_blank"} dejó de funcionar como **Subnet Router** y como **Exit Node**.

La causa eran dos problemas independientes: Tailscale intentaba usar `iptables` *legacy* en un sistema que usa `nftables`, y el IPv6 estaba desactivado directamente en el kernel.

El objetivo era doble:

* Acceder desde un cliente de mi *Tailnet* (por ejemplo, el móvil conectado a 4G/5G) a los servicios de mi red `192.168.1.0/24`
* Usar la Raspberry Pi 4 como *Exit Node* para salir a Internet a través de la conexión de casa

En este artículo explico los síntomas, las comprobaciones realizadas para encontrar la causa y la solución aplicada.

## Configuración de partida

La configuración inicial del contenedor era aparentemente correcta: modo *kernel* (no *userspace*), acceso a `/dev/net/tun`, capacidad `NET_ADMIN` y red `host` para poder enrutar tráfico.

El fichero `compose.yaml` era el siguiente:

```yaml
services:
  tailscale:
    image: tailscale/tailscale:v1.102.3
    container_name: tailscale
    hostname: tailscale
    restart: unless-stopped
    cap_add:
      - NET_ADMIN
    devices:
      - /dev/net/tun:/dev/net/tun
    environment:
      TZ: ${TZ:-Europe/Madrid}
      TS_ENABLE_HEALTH_CHECK: true
      TS_ENABLE_METRICS: false
      TS_HOSTNAME: rpi4
      TS_ROUTES: ${HOMELAB_NETWORK:-192.168.1.0/24}
      TS_STATE_DIR: /var/lib/tailscale
      TS_USERSPACE: false
      TS_EXTRA_ARGS: --advertise-exit-node
    # Network mode 'host' para gestionar interfaces de red
    # y enrutar tráfico externo (exit node)
    network_mode: host
    volumes:
      - /usr/share/zoneinfo/${TZ:-Europe/Madrid}:/etc/localtime:ro
      - ./volumes/state:/var/lib/tailscale
```

Y el fichero `.env`:

```
TZ=Europe/Madrid
HOMELAB_NETWORK=192.168.1.0/24
```

En el sistema operativo ya existía el fichero `/etc/sysctl.d/99-tailscale.conf` para activar el *IP forwarding* y, para no usar IPv6 en casa, se había añadido `ipv6.disable=1` en `/boot/firmware/cmdline.txt`.

## Síntomas

Aunque las *Subnet routes* y el *Exit node* estaban aprobados en la consola de administración de Tailscale, en *Edit route settings* aparecía el aviso **Unable to relay traffic**, indicando que el equipo tenía el *IP forwarding* desactivado.

El comando `tailscale status` dentro del contenedor mostraba dos problemas en su *health check*:

```
# Health check:
#     - Subnet routing is enabled, but IP forwarding is disabled. Check that IP forwarding is enabled on your machine.
#     - running [/usr/sbin/iptables -t filter -N ts-input --wait]: exit status 3: modprobe: can't change directory to '/lib/modules': No such file or directory
iptables v1.8.11 (legacy): can't initialize iptables table `filter': Table does not exist (do you need to insmod?)
```

> **Nota**: Lo curioso es que el **IP forwarding** de IPv4 sí estaba activo (`net.ipv4.ip_forward = 1`), por lo que el primer aviso no tenía una explicación evidente.

## Análisis

Antes de cambiar nada, se recopiló información del *host* y del contenedor con los siguientes comandos:

```bash
# Kernel, TUN y sysctl
uname -r
lsmod | grep -w tun ; ls -l /dev/net/tun
sysctl net.ipv4.ip_forward net.ipv6.conf.all.forwarding \\
       net.ipv4.conf.all.rp_filter net.ipv4.conf.default.rp_filter
grep -rsE 'forward|rp_filter' /etc/sysctl.conf /etc/sysctl.d/

# Firewall
systemctl is-active nftables ufw firewalld
update-alternatives --query iptables | grep -i value
sudo nft list tables
sudo nft list ruleset | grep -nE 'chain (FORWARD|ts-)|policy|tailscale0'

# Red
ip -br addr
ip rule
ip route show table 52

# Docker y contenedor
docker logs tailscale 2>\&1 | grep -iE 'firewall|nftables|iptables|forward|warn|error'
docker exec tailscale tailscale status
docker exec tailscale tailscale debug prefs | grep -E 'AdvertiseRoutes|NetfilterMode|NoSNAT'
```

Los resultados más relevantes fueron:

|Comprobación|Resultado|Valoración|
|-|-|-|
|Módulo `tun` y `/dev/net/tun`|Cargado y accesible|Correcto|
|`net.ipv4.ip_forward`|`1`|Correcto|
|`net.ipv6.conf.all.forwarding`|`cannot stat /proc/sys/net/ipv6/...`|**Problema**|
|`rp_filter`|`all = 0`, `default = 2` (modo *loose*)|Correcto|
|Backend de `iptables` en el *host*|`/usr/sbin/iptables-nft`|Host con `nftables`|
|Política de la cadena `FORWARD`|`accept`|Correcto|
|Cadenas `ts-forward` / `ts-postrouting`|No existen|**Problema**|
|Modo de firewall de Tailscale|`default choosing iptables` (*legacy*)|**Problema**|
|`ip rule` y tabla `52`|Reglas de Tailscale presentes|Correcto|

> **Nota**: La política de `FORWARD` estaba en `accept` porque el **forwarding** ya estaba activo cuando arrancó Docker. Si Docker lo hubiera activado por sí mismo, habría cambiado la política a `drop`.

Para descartar reglas añadidas a mano durante pruebas anteriores, se reinició la Raspberry Pi y se comprobó que no había reglas persistentes (`netfilter-persistent` no instalado y servicio `nftables` desactivado). Los resultados fueron los mismos.

## Causa 1: `iptables` legacy frente a `nftables`

Tailscale elegía por defecto el modo `iptables` y usaba el binario *legacy* incluido en su imagen. Debian 13 trabaja con `nftables` (a través de `iptables-nft`), por lo que las tablas *legacy* no existen. El contenedor tampoco puede cargar módulos del kernel porque no tiene acceso a `/lib/modules`.

En los registros se veía claramente:

```
router: default choosing iptables
wgengine: Reconfig: router config failed (running [/usr/sbin/iptables -t filter -N ts-input --wait]: exit status 3 ...
iptables v1.8.11 (legacy): can't initialize iptables table `filter': Table does not exist
```

La consecuencia es que Tailscale **no podía crear sus cadenas** `ts-input`, `ts-forward` y `ts-postrouting`, y sin esta última **no hay *masquerade* (SNAT)**. Los paquetes del móvil llegaban a la LAN con su IP de origen `100.x.x.x`. Los equipos respondían a través del router de casa, que no sabe cómo llegar a la red `100.64.0.0/10`, y la conexión nunca se completaba.

### Solución

Forzar el modo `nftables` mediante la variable `TS_DEBUG_FIREWALL_MODE` en el `compose.yaml`:

```yaml
    environment:
      TS_DEBUG_FIREWALL_MODE: nftables
```

Y recrear el contenedor (un simple reinicio no aplica los cambios del entorno):

```bash
docker compose up -d
```

Tras el cambio, los registros confirman el nuevo modo:

```
envknob: TS_DEBUG_FIREWALL_MODE="nftables"
router: netfilter running in nftables mode, v6 = false
```

Y las cadenas de Tailscale ya aparecen en las tablas `ip filter` e `ip nat`, incluida la regla de *masquerade*:

```
chain ts-postrouting {
        meta mark \& 0x00ff0000 == 0x00040000 counter packets 0 bytes 0 masquerade
}
```

## Causa 2: `ipv6.disable=1` en el kernel

El aviso de *IP forwarding* desactivado se mantenía incluso con el modo `nftables`. La causa era el IPv6, como confirmaron estos comandos:

```bash
tr ' ' '\\n' < /proc/cmdline | grep -i ipv6
```

```
ipv6.disable=1
```

El parámetro `ipv6.disable=1` **elimina la pila IPv6 completa del kernel**, hasta el punto de que el directorio `/proc/sys/net/ipv6/` ni siquiera existe. Como *Exit Node*, Tailscale anuncia también la ruta `::/0` y comprueba el valor de `net.ipv6.conf.all.forwarding`. Al no encontrar el fichero, concluye que el *forwarding* está desactivado y lo notifica a la consola.

### Solución

El equivalente en Linux a desmarcar IPv6 en la tarjeta de red de Windows es mantener la pila cargada pero desactivar IPv6 en las interfaces mediante `sysctl`. Así ninguna interfaz recibe dirección IPv6, pero el fichero de *forwarding* existe y puede valer `1`.

1. Quitar el parámetro del arranque (el fichero debe quedar en una sola línea):

```bash
   sudo cp /boot/firmware/cmdline.txt /boot/firmware/cmdline.txt.bak
   sudo sed -i 's/ **ipv6\\.disable=1//' /boot/firmware/cmdline.txt
   ```

2. Añadir las siguientes líneas en `/etc/sysctl.d/99-tailscale.conf`:

```
   net.ipv6.conf.all.disable_ipv6=1
   net.ipv6.conf.default.disable_ipv6=1
   net.ipv6.conf.all.forwarding=1
   ```

3. Desactivar IPv6 también en NetworkManager, que controla el valor de `disable_ipv6` de cada interfaz y podría volver a activarlo en `eth0`:

```bash
   nmcli -t -f NAME,DEVICE con show --active | grep eth0
   sudo nmcli con mod "<NOMBRE>" ipv6.method disabled
   ```

4. Reiniciar la Raspberry Pi.

Tras el reinicio, `ip -6 addr` no devuelve nada y Tailscale detecta la configuración correctamente:

```
router: disabling tunneled IPv6 due to system IPv6 config: disable_ipv6 is set
```

> **Nota**: El **Exit Node** funciona únicamente con IPv4, que es lo esperado en una red sin IPv6. Unbound ya tenía configurado `do-ip6: no`.

## Ajustes adicionales

Durante el análisis aparecieron otros avisos menores que también se corrigieron.

### `src_valid_mark`

Con `rp_filter` en modo *loose*, Tailscale activa un mecanismo para compensarlo que requiere `net.ipv4.conf.all.src_valid_mark=1`. El contenedor no puede escribirlo porque `/proc/sys` es de solo lectura:

```
router: warning: failed to enable src_valid_mark: ... read-only file system
```

Como el contenedor usa la red del *host*, basta con configurarlo en `/etc/sysctl.d/99-tailscale.conf`. El aviso sigue apareciendo en los registros, pero es inofensivo porque el valor ya es `1`:

```bash
docker exec tailscale cat /proc/sys/net/ipv4/conf/all/src_valid_mark
```

### UDP GRO

Tailscale avisaba de que el GRO de `eth0` no era óptimo para reenviar tráfico UDP:

```
Warning: UDP GRO forwarding is suboptimally configured on eth0
```

Para aplicarlo en cada arranque se crea un script de NetworkManager directamente en `/etc/NetworkManager/dispatcher.d/` (no en sus subdirectorios). El nombre es libre, pero debe ser propiedad de `root` y ejecutable:

```bash
sudo tee /etc/NetworkManager/dispatcher.d/50-tailscale-gro >/dev/null <<'EOF'
#!/bin/sh
[ "$1" = "eth0" ] \&\& [ "$2" = "up" ] \&\& /usr/sbin/ethtool -K eth0 rx-udp-gro-forwarding on rx-gro-list off
EOF
sudo chmod 755 /etc/NetworkManager/dispatcher.d/50-tailscale-gro
```

### *Healthcheck* de Docker

La variable `TS_ENABLE_HEALTH_CHECK` activa el *endpoint* `/healthz` en el puerto `9002`, pero Docker no lo usa si no se define un `healthcheck`. La imagen está basada en Alpine e incluye `wget`:

```yaml
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://127.0.0.1:9002/healthz"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 20s
```

> **Nota**: El **widget** de Tailscale para [Homepage](https://gethomepage.dev/widgets/services/tailscale/) no usa este **endpoint**, sino la API de Tailscale con un **token** y el ID del dispositivo.

### Zona horaria

El contenedor mostraba la hora en UTC a pesar de tener `TZ=Europe/Madrid` y `/etc/localtime` montado. Cuando `TZ` está definida, tiene prioridad sobre `/etc/localtime`, pero la imagen no incluye la base de zonas horarias (`tzdata`). Al no encontrar `/usr/share/zoneinfo/Europe/Madrid`, se usaba UTC.

La solución es montar la base de zonas horarias del *host*:

```yaml
    volumes:
      - /usr/share/zoneinfo:/usr/share/zoneinfo:ro
```

## Configuración final y verificación

El fichero `compose.yaml` definitivo es el siguiente:

```yaml
services:
  tailscale:
    image: tailscale/tailscale:v1.102.3
    container_name: tailscale
    hostname: tailscale
    restart: unless-stopped
    cap_add:
      - NET_ADMIN
    devices:
      - /dev/net/tun:/dev/net/tun
    environment:
      TZ: ${TZ:-Europe/Madrid}
      TS_DEBUG_FIREWALL_MODE: nftables
      TS_ENABLE_HEALTH_CHECK: "true"
      TS_ENABLE_METRICS: "false"
      TS_HOSTNAME: rpi4
      TS_ROUTES: ${HOMELAB_NETWORK:-192.168.1.0/24}
      TS_STATE_DIR: /var/lib/tailscale
      TS_USERSPACE: "false"
      TS_EXTRA_ARGS: --advertise-exit-node
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://127.0.0.1:9002/healthz"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 20s
    # Network mode 'host' para gestionar interfaces de red
    # y enrutar tráfico externo (exit node)
    network_mode: host
    volumes:
      - /usr/share/zoneinfo:/usr/share/zoneinfo:ro
      - ./volumes/state:/var/lib/tailscale
```

Y el fichero `/etc/sysctl.d/99-tailscale.conf`:

```
# Subnet y Exit Node (Tailscale)
net.ipv4.ip_forward=1
net.ipv4.conf.all.src_valid_mark=1
# IPv6: pila cargada pero sin direcciones; forwarding activo para Tailscale
net.ipv6.conf.all.disable_ipv6=1
net.ipv6.conf.default.disable_ipv6=1
net.ipv6.conf.all.forwarding=1
```

Con esta configuración, `tailscale status` ya no muestra ningún problema en el *health check*, el contenedor aparece como `healthy` y el aviso **Unable to relay traffic** desaparece de la consola de Tailscale.

### Pruebas desde el móvil

Con el móvil conectado a 4G/5G (fuera de la Wi-Fi de casa) y Tailscale activo:

* **Subnet Router**: acceso correcto a Proxmox, a AdGuard Home en la Raspberry Pi y a AdGuard Home en el LXC de Proxmox
* **Exit Node**: sin activarlo, [ipinfo.io](https://ipinfo.io/json) muestra la IP de la red móvil. Al seleccionar `rpi4` como *Exit Node*, muestra la IP pública de la conexión de casa

Durante las pruebas se puede comprobar en la Raspberry Pi cómo aumentan los contadores de las reglas de Tailscale:

```bash
sudo nft list ruleset | grep -E 'chain ts-(forward|postrouting)' -A4
```

## Referencias

* [Tailscale](https://tailscale.com/)
* [IP forwarding](https://tailscale.com/s/ip-forwarding), Tailscale
* [Homepage: Tailscale widget](https://gethomepage.dev/widgets/services/tailscale/)
* [Instalar AdGuard Home y Unbound en Docker](https://www.manelrodero.com/blog/instalar-adguard-home-y-unbound-en-docker), manelrodero.com

### Historial de cambios

* **2026-10-20**: Documento inicial (creado por Claude)
