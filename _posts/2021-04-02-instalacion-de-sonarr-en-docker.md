---
layout : post
blog-width: true
title: 'Instalación de Sonarr en Docker'
date: '2021-04-02 20:09:35'
last-updated: '2024-10-06 09:30:50'
published: true
tags:
- Docker
author:
  display_name: Manel Rodero
---

[Sonarr](https://sonarr.tv/) es un PVR para usuarios de Usenet y BitTorrent. Puede monitorizar múltiples feeds RSS para encontrar nuevos episodios de una serie, descargarlos, clasificarlos y cambiarles el nombre de forma automática.

También se puede configurar para actualizar automáticamente la calidad de los archivos ya descargados cuando esté disponible un formato de mejor calidad.

La [instalación en Docker](https://hub.docker.com/r/linuxserver/sonarr) se realiza usando la imagen `linuxserver/sonarr`.

La forma más sencilla es usar un fichero `docker-compose.yml` con el siguiente contenido:

```yaml
version: '3'
services:
  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    container_name: sonarr
    environment:
      - PUID=1001
      - PGID=115
      - TZ=Europe/Madrid
    volumes:
      - /home/pi/volumes/sonarr:/config
      - /data/media/tvseries:/data/tvseries
      - /data/torrents:/data/torrents
    ports:
      - 8989:8989
    restart: unless-stopped
```

A continuación, se puede ejecutar el comando `docker-compose up -d` o usar el contenido del fichero en Portainer.

# Configuración

Una vez en marcha, se puede acceder a Sonarr a través del puerto `8989` (en este ejemplo [http://192.168.1.180:8989](http://192.168.1.180:8989)) y comenzar la configuración.

Lo primero, y más importante, es crear una contraseña para el usuario administrador:

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
  * Category: sonarr
  * Remove completed: Enable
  * Test > Save
  
Para encontrar los capítulos hay que añadir uno o más **indexer** de Jackett haciendo lo siguiente:

* Settings > Indexers > Add > Torznab > Custom
  * Name: Jackett
  * URL &rarr; Copiarla desde Jackett
  * API Key &rarr; Copiarla desde Jackett
  * Ajustar las categorías

## Volúmenes comunes

Es recomendable utilizar un [volumen común](https://sonarr.tv/#downloads-v3-docker) **dentro** de los contenedores para evitar problemas con los _fast moves_ y los _hard links_.

Por ejemplo:

* `/tvseries` &rarr; `/data/tvseries`
* `/torrents` &rarr; `/data/torrents`

# Actualizar

Si ya se había instalado Sonarr anteriormente, se puede actualizar de la siguiente manera:

```
docker stop sonarr
docker rm sonarr
docker rmi lscr.io/linuxserver/sonarr
docker-compose up -d
```

# Soporte

Algunos comandos para gestionar la configuración del contenedor:

```
# Acceder al shell mientras el contenedor está ejecutándose
docker exec -it sonarr /bin/bash

# Monitorizar los logs del contenedor en tiempo real
docker logs -f sonarr
```

# Referencias

* [Ultimate Media Server : Episode 3 - Configuring Jackett, Sonarr, and Radarr](https://youtu.be/uvc4TnhVecA) @ DB Tech
* [Ultimate Raspberry Pi Server with Sonarr, Radarr, Jackett, NZBGet and Deluge on Docker](https://www.youtube.com/watch?v=oLxsSQIqOMw) @ LMDS GreenFrog
* [Les Tutos: Docker No. 9: Multimedia Automation Part3: Sonarr - Séries](https://www.youtube.com/watch?v=_absmgualKM)
* [TRaSH Guides: Sonarr](https://trash-guides.info/Sonarr/)
* [WikiArr: Sonarr](https://wiki.servarr.com/sonarr)

### Historial de cambios

* **2022-12-03**: Revisión del documento y corrección de errores.
* **2023-01-15**: Cambio a _path_ absolutos.
* **2024-10-06**: Adaptación formato cambios.
