---
layout : post
blog-width: true
title: 'Usar NordVPN con qBittorrent en Proxmox LXC'
date: '2025-03-25 17:40:36'
#last-updated: '2025-03-25 17:40:36'
published: true
tags:
- Proxmox
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2025-03-25_cover.png"
thumbnail-img: ""
---

En este artículo detallaré cómo he configurado la [instalación de qBittorrent en Proxmox LXC](instalar-qbittorrent-en-proxmox-lxc) que hice en Noviembre de 2023 para que utilice **NordVPN**, un servicio de red privada virtual que se utiliza para proteger la privacidad y la seguridad en línea.

# NordVPN

[NordVPN](https://nordvpn.com/download/linux/){:target="_blank"} funciona cifrando la conexión a Internet y redirigiéndola a través de servidores remotos ubicados en diferentes países, lo cual tiene varias ventajas:

* **Privacidad**: Oculta la dirección IP y ubicación, ayudando a navegar de forma anónima
* **Seguridad**: Cifra el tráfico para protegerlo de _hackers_, especialmente en redes Wi-Fi públicas
* **Acceso global**: Permite acceder a contenido restringido geográficamente al simular estar en otro país
* **Prevención de rastreo**: Evita que el proveedor de Internet o anunciantes sigan las actividades en línea

NordVPN también incluye algunas funciones avanzadas, como el **protocolo NordLynx** basado en **WireGuard** para conexiones rápidas o un **Kill Switch** para protegernos en caso de interrupción de la VPN.

## Instalación

Existen muchos tutoriales que configuran NordVPN y qBittorrent usando los contenedores Docker [`linuxserver/qbittorrent`](https://github.com/linuxserver/docker-qbittorrent){:target="_blank"} de LinuxServer.io y [`bubuntux/nordvpn`](https://github.com/bubuntux/nordvpn){:target="_blank"} de Bubuntux.

En mi caso utilizaré una idea diferente, muy similar a la descrita en el artículo "[How to use NordVPN in a LXD container](https://blog.simos.info/how-to-use-nordvpn-in-a-lxd-container/){:target="_blank"}", para instalar NordVPN en el mismo contenedor LXC donde ya había instalado qBittorrent anteriormente.

Seguiré las instrucciones oficiales del artículo "[Installing NordVPN on Linux distributions](https://support.nordvpn.com/hc/en-us/articles/20196094470929-Installing-NordVPN-on-Linux-distributions){:target="_blank"}" para tener de forma nativa características como la **conexión o el Kill Switch automáticos**.

Crearemos un directorio para descargar el _script_ de instalación y lo ejecutaremos **después de revisarlo**:

```bash
mkdir ~/nordvpn && cd ~/nordvpn
curl -sSf https://downloads.nordcdn.com/apps/linux/install.sh > install-nordvpn.sh
sh install-nordvpn.sh
```

Este _script_, preparado para ejecutarse en diferente distribuciones de Linux como la Debian de mi LXC, se encarga de:

1. Añadir la clave del repositorio de NordVPN (`/etc/apt/trusted.gpg.d/nordvpn_public.asc`)
2. Añadir el repositorio de NordVPN (`/etc/apt/sources.list.d/nordvpn.list`)
3. Instalar NordVPN (`apt update` & `apt install nordvpn`)

## Configuración

Para [**iniciar sesión en NordVPN desde CLI**](https://support.nordvpn.com/hc/en-us/articles/20226600447633-How-to-log-in-to-NordVPN-on-Linux-devices-without-a-GUI){:target="_blank"} es necesario obtener un **token** desde nuestra [cuenta de NordVPN](https://my.nordaccount.com/){:target="_blank"}:

* Acceder a **Services** &rarr; **NordVPN**
* Localizar la pestaña **Access Token**
* Pulsar el botón `Generate new token`
* Marcar la opción para que token no expire en 30 días
* Hacer clic en `Copy and close`

Con este token se podrá iniciar sesión desde la CLI:

```bash
nordvpn logout --persist-token > /dev/null
nordvpn login --token xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

A partir de la versión **3.18.4** de la aplicación de NordVPN para Linux, el **tráfico de red desde la LAN** a través de un ordenador conectado a la VPN **está bloqueado por defecto**.

Para desbloquear el tráfico LAN, hay que activar el descubrimiento de LAN (`LAN discovery`) o añadir a la lista permitida (`allowlist`) cualquier subred o tráfico IP que deba ser redirigido.

En mi caso, añadiré la subred de mi LAN para permitir el acceso a la interfaz web de qBittorrent desde todos los ordenadores de la misma:

```bash
nordvpn allowlist add subnet 192.168.1.0/24
```

A continuación se configura NordVPN para que utilice el [**protocolo NordLynx basado en WireGuard**](https://nordvpn.com/blog/nordlynx-protocol-wireguard/){:target="_blank"}:

```bash
nordvpn set technology NordLynx
```

También habilito el **Kill Switch** para que este contenedor LXC no tenga conectividad si se desconecta la VPN (ésto se consigue utilizando reglas de `iptables`):

```bash
nordvpn set killswitch enabled
```

Se cambia la configuración DNS para utilizar los servidores de NordVPN:

```bash
nordvpn set dns 103.86.96.100 103.86.99.10
```

Se deshabilitan el envío de telemetría (analíticas) a NordVPN:

```bash
nordvpn set analytics disable
```

Se configura la conexión automática para que, en caso de reiniciar el contenedor LXC, todo vuelva a la normalidad:

```bash
nordvpn set autoconnect enable
```

La configuración final es la siguiente:

```plaintext
Technology: NORDLYNX
Firewall: enabled
Firewall Mark: 0xe1f1
Routing: enabled
Analytics: disabled
Kill Switch: enabled
Threat Protection Lite: disabled
Notify: enabled
Tray: enabled
Auto-connect: enabled
IPv6: disabled
Meshnet: disabled
DNS: 103.86.96.100, 103.86.99.100
LAN Discovery: disabled
Virtual Location: enabled
Post-quantum VPN: disabled
Allowlisted subnets:
        192.168.1.0/24
```

## Conexión

Finalmente, se podrá usar el comando `nordvpn connect <country>` para conectarse a un determinado país. También se puede utilizar el comando `nordvpn connect --group <group> <country>` para conectarse a un grupo de servidores de un país determinado.

Si al conectar la VPN, tenemos errores relacionados con el dispositivo `tun`, seguramente es debido a que no se ha añadido correctamente al contenedor LXC desde Proxmox en el fichero `/etc/pve/lxc/<id>.conf`:

```plaintext
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```

## Comprobación

Para comprobar que todo está funcionando correctamente, se puede utilizar el comando `nordvpn status` para mostrar información sobre el servidor al que se ha conectado:

```plaintext
Status: Connected
Server: <Country> #<Number>
Hostname: <Country><Number>.nordvpn.com
IP: xxx.xxx.xxx.xxx
Country: <Country>
City: <City>
Current technology: NORDLYNX
Current protocol: UDP
Post-quantum VPN: Disabled
Transfer: 2.30 MiB received, 1.61 MiB sent
Uptime: 1 hour 12 minutes 43 seconds
```

También se puede utilizar un servicio externo como `ipv4.ipleak.net` para comprobar la dirección IP que nos ha asignado NordVPN:

```bash
curl https://ipv4.ipleak.net/json/
```

{: .box-note}
Tanto la `ip` como el `isp_name` deberían ser diferentes a los de nuestro proveedor de Internet. Si no es así, hay que revisar la configuración ;-)

## Actualización

El blog "[NordVPN for Linux: Release notes](https://nordvpn.com/blog/nordvpn-linux-release-notes){:target="_blank"}" siempre está actualizado con la información relativa a la última versión disponible.

La página de GitHub del [NordVPN Linux client](https://github.com/NordSecurity/nordvpn-linux/releases) también está actualizada con la última versión disponible.

Para actualizar, únicamente hay que usar los comandos `apt update && apt upgrade -y` y reiniciar el contenedor LXC para asegurarnos que todo carga correctamente.

# qBittorrent

Finalmente, es necesario hacer cambios en qBittorrent para que únicamente funcione a través de NordVPN (es decir, que utilice la interfaz **`nordlynx`** en lugar de `eth0`)

![Nordlynx interface][1]

{: .box-note}
También se puede configurar para que use el protocolo `SOCKS5` siguiendo el artículo "[NordVPN proxy setup for qBittorrent](https://support.nordvpn.com/hc/en-us/articles/20195967385745-NordVPN-proxy-setup-for-qBittorrent){:target="_blank"}".

## IPv4 Forwarding

Si se necesita que todos los dispositivos de la LAN puedan salir a Internet a través de NordVPN, tal como se explica en el vídeo "[¡La VPN Definitiva para Cada Rincón de tu Casa!](https://www.youtube.com/watch?v=BSRjnJ7yIIg){:target="_blank"}", entonces es necesario habilitar el **IPv4 forwarding** a traves del comando `sysctl`.

Para ello hay que editar el fichero `/etc/sysctl.conf`, configurar `net.ipv4.ip_forward=1` y aplicar los cambios usando el comando `sysctl -p`.

En algún sitio he leído que también es necesario añadir esta regla `iptables -t nat -A POSTROUTING -o tun0 -j MASQUERADE` al _firewall_ del contenedor LXC. Pero, como esta vía no la he explorado mucho, no sabría decir si es necesario hacer algo más.

# Referencias

* [Configuración de un contenedor con NordVPN para usarlo de punto de acceso a Internet de otros](https://kslcsdalsadg.github.io/docker-puente-nordvpn/){:target="_blank"} by John Doe
* [qBittorrent client with NordVPN integration using Docker](https://github.com/barbar0jav1er/qbittorrent-nordvpn){:target="_blank"} by Barbar0Jav1er
* [Official qbittorrent-nox docker image](https://github.com/qbittorrent/docker-qbittorrent-nox){:target="_blank"} by qBittorrent
* [How to build the NordVPN Docker image](https://support.nordvpn.com/hc/en-us/articles/20465811527057-How-to-build-the-NordVPN-Docker-image
){:target="_blank"} by NordVPN
* [Running docker inside an unprivileged LXC container on Proxmox](https://du.nkel.dev/blog/2021-03-25_proxmox_docker/){:target="_blank"} by du.nkel.dev
* [Proxmox: Run Docker on Linux Containers (LXC)](https://benheater.com/proxmox-run-docker-on-linux-containers-lxc/) by 0xBEN


### Historial de cambios

* **2025-03-25**: Documento inicial

[1]: /assets/img/blog/2025-03-25_image_1.png "Nordlynx interface"
