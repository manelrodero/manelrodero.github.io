---
layout : post
blog-width: true
title: 'Instalación de Lidarr en Docker'
date: '2021-07-27 22:43:58'
published: true
tags:
- Docker
author:
  display_name: Manel Rodero
---

#### _**Actualizaciones**:_

* **2023-01-15**: Revisión del documento y corrección de errores.
* **2023-01-15**: Cambio a _path_ absolutos.

# Instalación

[Lidarr](https://lidarr.audio/) es un gestor de la colección de música para usuarios de Usenet y BitTorrent. Puede monitorizar múltiples feeds RSS para encontrar nuevos álbumes de los artistas favoritos, descargarlos, clasificarlos y cambiarles el nombre de forma automática.

También se puede configurar para actualizar automáticamente la calidad de los archivos ya descargados cuando esté disponible un formato de mejor calidad.

La [instalación en Docker](https://hub.docker.com/r/linuxserver/lidarr) se realiza usando la imagen `linuxserver/lidarr`.

La forma más sencilla es usar un fichero `docker-compose.yml` con el siguiente contenido:

```yaml
version: '3'
services:
  lidarr:
    image: lscr.io/linuxserver/lidarr:latest
    container_name: lidarr
    environment:
      - PUID=1001
      - PGID=115
      - TZ=Europe/Madrid
    volumes:
      - /home/pi/volumes/lidarr:/config
      - /data/media/music:/data/music
      - /data/torrents:/data/torrents
    ports:
      - 8686:8686
    restart: unless-stopped
```

Se puede usar `docker-compose up -d` o usar el contenido del fichero en Portainer.

# Configuración

Una vez en marcha, se puede acceder a Lidarr a través del puerto `8686` (en este ejemplo [http://192.168.1.180:8686](http://192.168.1.180:8686)) y comenzar la configuración.

Lo primero y más importante es crear una contraseña para el usuario administrador:

* Settings > General > Authentication > Forms (Login page)
* Settings > General > Username > `admin`
* Settings > General > Password > `*********`
* Settings > General > Analytics > Disable Send Anonymous Usage Data

A continuación se graban los cambios y se reinicia la aplicación cuando ésta lo indique.

Para que la aplicación pueda descargar las canciones de los álbumes, se debe configurar un cliente Bittorrent:

* Settings > Download clients > Add
  * Torrent > Deluge
  * Name: Deluge
  * Host: 192.168.1.180
  * Password: `**********`
  * Category: lidarr
  * Test > Save
  
Para encontrar las canciones hay que añadir uno o más **indexer** de Jackett haciendo lo siguiente:

* Settings > Indexers > Add > Torznab > Custom
  * Name: Jackett
  * URL &rarr; Copiarla desde Jackett
  * API Key &rarr; Copiarla desde Jackett
  * Ajustar las categorías

## Volúmenes comunes

Es recomendable utilizar un [volumen común](https://lidarr.audio/#downloads-v1-docker) **dentro** de los contenedores para evitar problemas con los _fast moves_ y los _hard links_.

Por ejemplo:

* `/music` &rarr; `/data/music`
* `/torrents` &rarr; `/data/torrents`

# Actualizar

Si ya se había instalado Lidarr anteriormente, se puede actualizar de la siguiente manera:

```
docker stop lidarr
docker rm lidarr
docker rmi lscr.io/linuxserver/lidarr
docker-compose up -d
```

# Soporte

Algunos comandos para gestionar la configuración del contenedor:

```
# Acceder al shell mientras el contenedor está ejecutándose
docker exec -it lidarr /bin/bash

# Monitorizar los logs del contenedor en tiempo real
docker logs -f lidarr
```

# Referencias

* [Ultimate Media Server : Episode 3 - Configuring Jackett, Lidarr, and Radarr](https://youtu.be/uvc4TnhVecA) @ DB Tech
* [Ultimate Raspberry Pi Server with Lidarr, Radarr, Jackett, NZBGet and Deluge on Docker](https://www.youtube.com/watch?v=oLxsSQIqOMw) @ LMDS GreenFrog
* [WikiArr: Lidarr](https://wiki.servarr.com/lidarr)
