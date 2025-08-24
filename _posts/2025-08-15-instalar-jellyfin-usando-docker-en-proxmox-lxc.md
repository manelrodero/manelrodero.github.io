---
layout : post
blog-width: true
title: 'Instalar Jellyfin usando Docker en Proxmox LXC'
date: '2025-08-15 20:22:11'
#last-updated: '2025-08-15 20:22:11'
published: true
tags:
- Proxmox
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2025-08-15_cover.png"
thumbnail-img: ""
---

Hace poco menos de una año, [instalé Jellyfin en Proxmox LXC](instalar-jellyfin-en-proxmox-lxc) usando los [Proxmox VE Helper-Scripts](https://community-scripts.github.io/ProxmoxVE/){:target="_blank"}.

Además, [habilité la Intel HD Graphics 630 en Proxmox](habilitar-intel-hd-graphics-630-en-proxmox) para activar la **aceleración hardware** que permite transcodificar usando `Intel QuickSync (QSV)`.

En este post se irá un poco más allá para combinar el **rendimiento** de los LXC con la **flexibilidad** de Docker a la hora de ejecutar la última versión de una aplicación.

Aprovechando la [plantilla de LXC con Debian 12 para ejecutar Docker en Proxmox](plantilla-de-lxc-con-debian-12-para-ejecutar-docker-en-proxmox) que hice hace unos días, crearé un nuevo contenedor Linux para ejecutar la [imagen oficial `jellyfin/jellyfin`](https://jellyfin.org/docs/general/installation/container){:target="_blank"}.

## Crear contenedor

Para crear el contenedor se siguen las instrucciones de la plantilla para obtener este código que se ejecuta en la _shell_ de Proxmox:

```bash
# Plantilla
ct_id="700"

# Datos del nuevo contenedor
new_ct_id="318"
lxcname="jellyfin"
ip="192.168.1.89/24"
gw="192.168.1.1"
cores="2"
memory="2048"
swap="512"
disk_size="5"

# Clonar la plantilla
pct clone "$ct_id" "$new_ct_id" --hostname "$lxcname" --full

# Añadir mountpoint para backups rsync
mkdir "/backups/rsync/$new_ct_id-$lxcname"
chown 100000:101000 "/backups/rsync/$new_ct_id-$lxcname"
chmod 770 "/backups/rsync/$new_ct_id-$lxcname"
pct set "$new_ct_id" -mp0 "/backups/rsync/$new_ct_id-$lxcname,mp=/mnt/rsync"

# (Opcional) Añadir mountpoint para media
# pct set "$new_ct_id" -mp1 "/datos/samba/media,mp=/mnt/samba"

# Añadir dispositivo /dev/dri/renderD128
GID=$(getent group render | cut -d: -f3)
echo "dev0: /dev/dri/renderD128,gid=$GID" >> "/etc/pve/lxc/$new_ct_id.conf"

# Dirección IP y puerta de enlace
pct set "$new_ct_id" -net0 name=eth0,bridge=vmbr0,ip="$ip",gw="$gw"

# Parámetros hardware
pct set "$new_ct_id" -cores "$cores" -memory "$memory" -swap "$swap"

# Tamaño del disco
pct resize "$new_ct_id" rootfs "${disk_size}G"

# Información del contenedor
pct config "$new_ct_id" 

# Iniciar contenedor
pct start "$new_ct_id"
```

## Jellyfin usando `docker compose`

La instalación es bastante sencilla siguiendo el ejemplo de la [documentación oficial](https://jellyfin.org/docs/general/installation/container){:target="_blank"}:

```yaml
services:
  jellyfin:
    image: jellyfin/jellyfin:10.10.7
    container_name: jellyfin
    user: 1000:1000
    group_add:
      # ID del grupo 'render'
      - "104"
    network_mode: 'host'
    environment:
      - TZ=Europe/Madrid
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - ./jellyfin_config:/config
      - ./jellyfin_cache:/cache
      - ./jellyfin_media:/local_media
    devices:
      # Dispositivo de renderización
      - /dev/dri/renderD128:/dev/dri/renderD128
    restart: 'unless-stopped'
```

Se crean los directorios necesarios:

```bash
mkdir ./jellyfin_config
mkdir ./jellyfin_cache
mkdir ./jellyfin_media
```

Se ejecuta el contenedor Docker:

```bash
docker compose up -d
```

Se accede a `http://192.168.1.89:8096/` y se realiza la configuración inicial de Jellyfin:

* Welcome to Jellyfin!
  * Preferred display language: `English`
* Tell us about yourself
  * Username: `admin`
  * Password: `********`
* Set up your media libraries
  * _Saltar este paso, de momento_
* Preferred Metadata Language
  * Language: **Spanish; Castilian**
  * Country/Region: **Spain**
* Set up Remote Access
  * Allow remote connections to this server: **No**
  * Enable automatic port mapping: **No**
* You’re Done!

A continuación se descarga el clásico [Big Buck Bunny](http://bbb3d.renderfarming.net/download.html){:target="_blank"} en formato 4K, 3840x2160, 60 fps.

La idea de usar este vídeo es probar, además de la descarga de metadatos y la reproducción desde la propia interfaz web, la transcodificación al solicitar un _bitrate_ más pequeño.

Se descarga el video en el subdirectorio `./jellyfin_media`:

```bash
mkdir -p "./jellyfin_media/Big Buck Bunny (2008)" && cd "./jellyfin_media/Big Buck Bunny (2008)"
curl -O "http://distribution.bbb3d.renderfarming.net/video/mp4/bbb_sunflower_2160p_60fps_normal.mp4"
```

Desde la página principal de la interfaz web (`Administration` &rarr; `Dashboard` &rarr; `Libraries`) se crea una nueva librería con la siguiente configuración:

* Add Media Library
  * Content type: `Movies`
  * Display name: `Local`
  * Folders: `/local_media`
  * `[X]` Enable the library
  * Preferred download language: **Spanish; Castilian**
  * Country/Region: **Spain**
  * Metadata savers: `Nfo`
  * Save artwork into media folders

Si todo funciona correctamente, Jellyfin obtendrá los **metadatos** de la película “Big Buck Bunny” desde los servidores configurados en la librería.

Se puede utilizar el botón ▶ en la parte superior derecha para reproducir la película. Se puede mostrar la información de reproducción usando la opción `Settings` &rarr; `Playback info` del reproductor de vídeo HTML de Jellyfin.

## Aceleración hardware

El servidor Jellyfin puede usar una tarjeta gráfica integrada o discreta (GPU) para realizar la [**transcodificación de video**](https://jellyfin.org/docs/general/post-install/transcoding/hardware-acceleration/){:target="_blank"} de manera muy eficiente sin forzar la CPU.

En mi caso, se utilizará **Intel Quick Sync Video (QSV)** en la tarjeta gráfica Intel HD Graphics 630 que incorpora el [Dell OptiPlex 7050 Micro](mini-pc-dell-optiplex-7050-micro) donde ejecuto [Proxmox VE 8.4.11](proxmox-ve-802-en-un-dell-optiplex-7050).

La imagen oficial de Docker incluye todos los [controladores multimedia Intel](https://jellyfin.org/docs/general/post-install/transcoding/hardware-acceleration/intel){:target="_blank"} necesarios para el modo usuario y el entorno de ejecución OpenCL.

Solo hay que pasar el **ID del grupo `render`** del host a Docker y modificar la configuración para que se ajuste a las necesidades de transcodificación que queramos.

```bash
getent group render | cut -d: -f3
```

```yaml
    group_add:
      # ID del grupo 'render'
      - "104"
```

También hay que pasar el **dispositivo de renderización** `/dev/dri/renderD128` de la GPU.

Este dispositivo permite **acceso restringido** únicamente a funciones de cómputo del hardware gráfico, incluyendo:

* Transcodificación de vídeo
* Decodificación por hardware (VAAPI, QSV)
* Procesamiento sin acceso al framebuffer

No es necesario pasar el **dispositivo principal** `/dev/dri/card0` de la GPU ya que permite **acceso completo** al hardware gráfico, incluyendo:

* Framebuffer (pantalla)
* Salida de vídeo
* Composición gráfica
* Aceleración por hardware

```yaml
    devices:
      # Dispositivo de renderización
      - /dev/dri/renderD128:/dev/dri/renderD128
```

Para que Jellyfin use la aceleración hardware, únicamente hay que configurarlo desde:

* `Administration` &rarr; `Dashboard` &rarr; `Playback` &rarr; `Transcoding`
  * Hardware acceleration: **Intel QuickSync (QSV)**

Si se reproduce la misma película y se fuerza la resolución a `420 kbps (360p)` para simular un dispositivo no compatible con el formato original de la película en la librería, se observa como Jellyfin realiza el **transcoding** al vuelo:

![Transcoding][1]

## (Opcional) Añadir `/mnt/samba`

Esto es una configuración opcional para poder compartir **media** entre diferentes LXC usando un servidor Samba.

En el LXC se añadió el _mountpoint_ `mp1`:

```plaintext
mp1: /datos/samba/media,mp=/mnt/samba
```

En el servidor Proxmox, el directorio `/datos/samba/media` tiene las siguientes protecciones:

```plaintext
drwxrwx--- 9 100000 100300 4096 Mar 25 21:40 media
```

En el LXC el directorio `/mnt/samba` tiene las siguientes protecciones (por ser _unprivileged_):

```plaintext
drwxrwx---  9 root sambashare 4096 Mar 25 21:40 samba
```

Se crea el grupo `sambashare` con GID `300` y se añade al usuario sin privilegios desde el que ejecuto los Dockers:

```plaintext
sambashare:x:300:manel
```

Este usuario sin privilegios es miembro de los siguientes grupos por la forma en que se creó la plantilla LXC:

```plaintext
uid=1000(manel) gid=1000(manel) groups=1000(manel),300(sambashare),996(docker)
```

### Historial de cambios

* **2025-08-15**: Documento inicial

[1]: /assets/img/blog/2025-08-15_image_1.png "Transcoding"
