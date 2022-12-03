---
layout : post
blog-width: true
title: 'Instalación de Radarr en Docker'
date: '2021-04-03 00:05:04'
published: true
tags:
- Docker
author:
  display_name: Manel Rodero
---

#### _**Actualizaciones**:_

* **2022-12-03**: Revisión del documento y corrección de errores.

# Instalación

[Radarr](https://radarr.video/) es un PVR para usuarios de Usenet y BitTorrent. Puede monitorizar múltiples feeds RSS para encontrar nuevas películas, descargarlas, clasificarlas y cambiarles el nombre de forma automática.

También se puede configurar para actualizar automáticamente la calidad de los archivos ya descargados cuando esté disponible un formato de mejor calidad.

La [instalación en Docker](https://hub.docker.com/r/linuxserver/radarr) se realiza usando la imagen `linuxserver/radarr`.

La forma más sencilla es usar un fichero `docker-compose.yml` con el siguiente contenido:

```yaml
version: '3'
services:
  radarr:
    image: lscr.io/linuxserver/radarr:latest
    container_name: radarr
    environment:
      - PUID=1001
      - PGID=115
      - TZ=Europe/Madrid
    volumes:
      - ~/volumes/radarr:/config
      - /data/media/movies:/data/movies
      - /data/torrents:/data/torrents
    ports:
      - 7878:7878
    restart: unless-stopped
```

A continuación, se puede ejecutar el comando `docker-compose up -d` o usar el contenido del fichero en Portainer.

# Configuración

Una vez en marcha, se puede acceder a Radarr a través del puerto `7878` (en este ejemplo [http://192.168.1.180:7878](http://192.168.1.180:7878)) y comenzar la configuración.

Lo primero, y más importante, es activar el login y crear una contraseña segura para el usuario administrador:

* Settings > General > Authentication > Forms (Login page)
* Settings > General > Username > `admin`
* Settings > General > Password > `*********`
* Settings > General > Analytics > Disable Send Anonymous Usage Data

A continuación se graban los cambios y se reinicia la aplicación cuando ésta lo indique.

Para que la aplicación pueda descargar los capítulos de las series, se debe configurar un cliente Bittorrent:

* Settings > Download clients > Add
  * Torrent > Deluge
  * Name: Deluge
  * Host: 192.168.1.180
  * Password: `**********`
  * Category: radarr
  * Remove completed: Enable
  * Test > Save

Para encontrar los capítulos hay que añadir uno o más **indexer** de Jackett haciendo lo siguiente:

* Settings > Indexers > Add > Torznab > Custom
  * Name: Jackett
  * URL &rarr; Copiarla desde Jackett
  * API Key &rarr; Copiarla desde Jackett
  * Ajustar las categorías

# Troubleshooting

## libseccomp2

> **Nota**: En la versión de 64-bits de Raspberry Pi OS no se ha encontrado este problema.

No se puede cargar WebUI en el puerto `7878`. Al probar el fichero de [LMDS with Docker on Raspberry Pi](https://greenfrognest.com/lmdsondocker.php) pasa exactamente lo mismo.

Entonces se ha buscado en Google y se ha llegado al [issue 126](https://github.com/linuxserver/docker-radarr/issues/126) y después al [issue 118](https://github.com/linuxserver/docker-radarr/issues/118).

Desde aquí se llega a la [FAQ de Radarr](https://wiki.servarr.com/Radarr_FAQ#I_am_using_a_Pi_and_Raspbian_and_Radarr_will_not_launch) y a la [FAQ de Linuxserver](https://docs.linuxserver.io/faq#my-host-is-incompatible-with-images-based-on-ubuntu-focal) donde se indica que el problema es debido a que Raspbian incluye una versión antigua de `libseccomp2`.

La versión incluida en Raspbian (una distribución de 32-bits basada en Debian Buster) es la **2.3.3.4-4** para `armhf` y no puede ejecutar un contenedor Docker basado en Ubuntu 20.04 (Ubuntu Focal).

Hay diferentes soluciones:

1. Usar una versión que no esté basada en Focal
2. Instalar de forma manual la [librería](http://ftp.us.debian.org/debian/pool/main/libs/libseccomp) usando `dpkg`
3. Añadir el [repositorio de _backports_ para Debian Buster](https://github.com/linuxserver/docker-jellyfin/issues/71#issuecomment-733621693)
4. Cambiar el SO (Ubuntu 20.04 arm64) o actualizarlo para que soporte esta librería (Raspberry Pi OS se puede actualizar a un kernel de 64 bits)

Antes de instalar la librería de forma manual (opción 2) tenemos la siguiente versión de `libseccomp2`:

```
pi@pi4nas:~ $ sudo apt search libseccomp2
Sorting... Done
Full Text Search... Done
libseccomp-dev/stable 2.3.3-4 armhf
  high level interface to Linux seccomp filter (development files)
```

Se descarga la librería y se instala:

```
wget http://ftp.us.debian.org/debian/pool/main/libs/libseccomp/libseccomp2_2.4.4-1~bpo10+1_armhf.deb
sudo dpkg -i libseccomp2_2.4.4-1~bpo10+1_armhf.deb
```

Después de la instalación:

```
pi@pi4nas:~ $ sudo apt search libseccomp2
Sorting... Done
Full Text Search... Done
libseccomp2/now 2.4.4-1~bpo10+1 armhf [installed,local]
  high level interface to Linux seccomp filter

```

## Volúmenes comunes

Es recomendable utilizar un [volumen común](https://radarr.video/#downloads-v3-docker) **dentro** de los contenedores para evitar problemas con los _fast moves_ y los _hard links_.

Por ejemplo:

* `/movies` &rarr; `/data/movies`
* `/torrents` &rarr; `/data/torrents`

# Actualizar

Si ya se había instalado Radarr anteriormente, se puede actualizar de la siguiente manera:

```
docker stop radarr
docker rm radarr
docker rmi ghcr.io/linuxserver/radarr
docker-compose up -d
```

# Soporte

Algunos comandos para gestionar la configuración del contenedor:

```
# Acceder al shell mientras el contenedor está ejecutándose
docker exec -it radarr /bin/bash

# Monitorizar los logs del contenedor en tiempo real
docker logs -f radarr
```

# Referencias

* [VPS Facil Radarr](https://vpsfacil.es/radarr/)
* [Ultimate Media Server : Episode 3 - Configuring Jackett, Sonarr, and Radarr](https://youtu.be/uvc4TnhVecA) @ DB Tech
* [Ultimate Raspberry Pi Server with Sonarr, Radarr, Jackett, NZBGet and Deluge on Docker](https://www.youtube.com/watch?v=oLxsSQIqOMw) @ LMDS GreenFrog
* [Les Tutos: Docker No. 9: Multimedia Automation Part2: Radarr - Films](https://www.youtube.com/watch?v=wwZ7o-eLvZg)
* [TRaSH Guides: Radarr](https://trash-guides.info/Radarr/)
* [WikiArr: Radarr](https://wiki.servarr.com/radarr)
