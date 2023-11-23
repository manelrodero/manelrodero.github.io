---
layout : post
blog-width: true
title: 'Instalar qBittorrent en Proxmox LXC'
date: '2023-11-05 13:05:29'
published: true
tags:
- Proxmox
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2023-11-05_cover.png"
thumbnail-img: ""  
---

Para seguir el proceso de migración de mi _Home Lab_ desde una pequeña [Raspberry Pi 4](instalar-raspberry-pi-os-64bits) a un [OptiPlex 7050 ejecutando Proxmox](proxmox-ve-802-en-un-dell-optiplex-7050) ahora le toca el turno a [**qBittorrent**](https://www.qbittorrent.org/), un cliente gratuito y fiable del protocolo P2P Bittorrent.

En este ocasion, en lugar de realizar la [instalación de qBittorrent en Docker](instalacion-de-qbittorrent-en-docker), se utilizará un contenedor ligero LXC de Proxmox con TurnKey Core. El procedimiento para crear este LXC es el mismo que usé para [ejecutar Docker en Proxmox LXC](docker-en-proxmox-lxc-con-turnkey-core).

# Características del LXC

Este **cliente de Bittorrent** con [qBittorrent](https://www.qbittorrent.org/) se creará sobre un LXC con las siguientes características:

* LXC no privilegiado, sin _nesting_ y sin `keyctl`
* CT ID 301
* Password + SSH key file
* Disco de **2GB**
* CPU con 1 _core_
* 1024MB de memoria
* IP fija 192.168.1.91

Después de crear el LXC, y antes de ponerlo en marcha, se añade el mismo punto de montaje que la [instalación de Samba](instalar-samba-en-proxmox-lxc):

```
# Añadir el punto de montaje al contenedor LXC (el directorio en LXC se crea automáticamente)
pct set 301 -mp0 /data/samba,mp=/mnt/samba
```

Esta instrucción añade la siguiente línea al archivo de configuración de esta máquina `/etc/pve/lxc/301.conf`:

```
mp0: /data/samba,mp=/mnt/samba
```

A continuación se pone en marcha el LXC desde la CLI de Proxmox y se procede a configurarlo de la misma manera que ya expliqué en el artículo sobre [LXC y Docker](docker-en-proxmox-lxc-con-turnkey-core):

```
pct start 301
```

# Acceder al LXC

Se puede acceder al LXC utilizando la opción `Console` de la GUI o, mucho mejor, mediante **SSH** gracias a la configuración de las claves públicas que se hizo en el mismo:

```Bash
ssh root@192.168.1.91
```

# Instalación de qBittorrent

La instalación de qBittorrent en una distribución basada en Debian es muy sencilla usando el comando `apt`. En este caso se instalará **qBittorrent-nox** que está especialmente diseñado para sistemas _headless_ y tiene una interfície web accesible en `http://<ip>:8080`:

```Bash
apt update
apt install qbittorrent-nox -y
```

## Usuario sin privilegios

Para aumentar la seguridad, se ejecutará qBittorrent-nox con un usuario sin privilegios:

```
adduser --system --uid 200 --group qbittorrent-nox
```

{: .box-note}
**Nota**: Se utiliza el UID 200 y el GID 200 para que no haya solapamiento con otros usuarios del _host_ o de otros contenedores LXC (p.ej. el de Samba).

A continuación se crea un grupo `sambashare` con el mismo GID 111 que tiene en el LXC de Samba y se añade al usuario `qbittorrent-nox` para no tener problemas con los permisos de los directorios montados bajo `/mnt/samba`:

```Bash
addgroup --system --gid 111 sambashare
adduser qbittorrent-nox sambashare
```

## Unidad de servicio de `systemd`

Para ejecutar qBittorrent-nox como servicio, es necesario crear una unidad de servicio de `systemd`. Para ello se crea el fichero `/etc/systemd/system/qbittorrent-nox.service`:

```Bash
nano /etc/systemd/system/qbittorrent-nox.service
```

Con el siguiente contenido:

```
[Unit]
Description=qBittorrent Command Line Client
After=network.target

[Service]
Type=forking
User=qbittorrent-nox
Group=sambashare
UMask=007
ExecStart=/usr/bin/qbittorrent-nox -d --webui-port=8080
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

{: .box-note}
**Nota**: Se establece que la [umask](https://es.wikipedia.org/wiki/Umask) de los ficheros y directorios creados por `qbittorrent-nox` sea **007** (lectura (4), escritura (2) y ejecución (1) disponibles para el usuario y el grupo). Esto hace que los ficheros creados tengan los permisos `rw-rw----` y los directorios `rwxrwx---`.

Finalmente se reinicia el _daemon_ de `systemd` y se pone en marcha el servicio `qbittorent-nox` asegurándose que se habilita en el inicio:

```
systemctl daemon-reload
systemctl start qbittorrent-nox
systemctl enable qbittorrent-nox
```

# Configuración

Una vez en marcha, se puede acceder a qBittorrent a través del puerto **8080** (en este ejemplo [http://192.168.1.91:8080](http://192.168.1.91:8080)) y comenzar la configuración.

Lo primero y más importante es cambiar la contraseña de acceso para el usuario `admin` (por defecto es `adminadmin`):

* Acceder al menú `Tools > Options`
* Acceder a la pestaña `Web UI`
* En la sección `Authentication`:
  * **Password** &rarr; `********`

A continuación se configura el resto de opciones:

* Acceder a la pestaña `Downloads`
* En la sección `When adding a torrent`:
  * Marcar la opción **Delete .torrent files afterwards**
* En la sección general:
  * Marcar la opción **Pre-allocate disk space for all files**
* En la sección `Saving Management`:
  * **Default Save Path**: `/home/qbittorrent-nox/Downloads/` &rarr; `/mnt/samba/torrents/downloaded/`
  * **Keep incomplete torrents in**: `/home/qbittorrent-nox/Downloads/temp/` &rarr; `/mnt/samba/torrents/downloading/`
* En la sección `Automatically add torrents from`:
  * **Monitored Folder**: `/mnt/samba/torrents/incoming/`

Y todas las que se necesiten ;-)

# Prueba

La "prueba de fuego" es descargar la ISO de Ubuntu Server 22.04.3 LTS (`ubuntu-22.04.3-live-server-amd64.iso.torrent`):

* Acceder al menú `File > Add Torrent Link`
* Pegar la URL `https://releases.ubuntu.com/22.04/ubuntu-22.04.3-live-server-amd64.iso.torrent`
* Pulsar el botón `Download`

Si todo funciona correctamente, la descarga debería producirse rápidamente tal como se muestra en la siguiente captura de pantalla:

![Descarga de ISO en qBittorrent-nox][1]

[1]: /assets/img/blog/2023-11-05_image_1.png "Descarga de ISO en qBittorrent-nox"
