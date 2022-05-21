---
layout : post
blog-width: true
title: 'Instalación de Bazarr en Docker'
date: '2021-07-28 22:20:21'
published: true
tags:
- Docker
author:
  display_name: Manel Rodero
---

[Bazarr](https://www.bazarr.media/) es una aplicación auxiliar de [Sonarr](instalacion-de-sonarr-en-docker) y [Radarr](instalacion-de-radarr-en-docker) que gestiona y descarga subtítulos.

La [instalación en Docker](https://hub.docker.com/r/linuxserver/bazarr) se realiza usando la imagen `linuxserver/bazarr`.

La forma más sencilla es usar un fichero `docker-compose.yml` con el siguiente contenido:

```
  bazarr:
    image: ghcr.io/linuxserver/bazarr:latest
    container_name: bazarr
    environment:
      - PUID=1001
      - PGID=115
      - TZ=Europe/Madrid
    volumes:
      - /home/pi/volumes/bazarr:/config
      - /data/media/movies:/data/movies
      - /data/media/tvseries:/data/tvseries
    ports:
      - 6767:6767
    restart: unless-stopped
```

Se puede usar `docker-compose up -d` o usar el contenido del fichero en Portainer.

# Configuración

Una vez en marcha, se puede acceder a Lidarr a través del puerto `6767` (en este ejemplo http://192.168.1.180:6767/) y comenzar la configuración.

Lo primero y más importante es crear una contraseña para el usuario administrador:

* Settings > General > Authentication > Form
* Settings > General > Username > `admin`
* Settings > General > Password > `*********`
* Settings > General > Analytics > Disable

A continuación se graban los cambios y se reinicia la aplicación si ésta lo indica.

Para que la aplicación pueda descargar los subtítulos se deben configurar diferentes opciones:

* Settings > Languages
  * English
  * Spanish
* Settings > Providers
* Settings > Subtitles
  * Disable Upgrade Previously Downloaded Subtitles
* Settings > Sonarr > Enable
  * Address: 192.168.1.180
  * API Key
  * Download only monitored
* Settings > Radarr > Enable
  * Address: 192.168.1.180
  * API Key
  * Download only monitored

## Volúmenes comunes

Es recomendable utilizar un [volumen común](https://lidarr.audio/#downloads-v1-docker) **dentro** de los contenedores para evitar problemas con los _fast moves_ y los _hard links_.

Por ejemplo:

* `/movies` &rarr; `/data/movies`
* `/tvseries` &rarr; `/data/tvseries`

# Actualizar

Si ya se había instalado Lidarr anteriormente, se puede actualizar de la siguiente manera:

```
docker stop bazarr
docker rm bazarr
docker rmi ghcr.io/linuxserver/bazarr

# Únicamente si no se usa un 'stack' en Portainer
docker run -d \
  --name=bazarr \
  -e PUID=1001 \
  -e PGID=115 \
  -e TZ=Europe/Madrid \
  -p 6767:6767 \
  -v /home/pi/volumes/bazarr:/config \
  -v /data/media/movies:/data/movies \
  -v /data/media/tvseries:/data/tvseries \  
  --restart unless-stopped \
  ghcr.io/linuxserver/bazarr:latest
```

# Soporte

Algunos comandos para gestionar la configuración del contenedor:

```
# Acceder al shell mientras el contenedor está ejecutándose
docker exec -it bazarr /bin/bash

# Monitorizar los logs del contenedor en tiempo real
docker logs -f bazarr
```

# Referencias

* [Easy Automated Home Media Server: VPN, Radarr, Sonarr, Lidarr, Librarian in 10 Minutes](https://www.youtube.com/watch?v=5rtGBwBuzQE) @ Techo Dad Life
