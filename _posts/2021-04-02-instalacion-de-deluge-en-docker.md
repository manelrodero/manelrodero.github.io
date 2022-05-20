---
layout : post
blog-width: true
title: 'Instalación de Deluge en Docker'
date: '2021-04-02 23:45:18'
published: true
tags:
- Docker
author:
  display_name: Manel Rodero
---

[Deluge](https://deluge-torrent.org/) es un cliente Bittorrent ligero, gratuito y multiplataforma.

La [instalación en Docker](https://hub.docker.com/r/linuxserver/deluge) se realiza usando la imagen `linuxserver/deluge`.

La forma más sencilla es usar un fichero `docker-compose.yml` con el siguiente contenido:

```
  deluge:
    image: ghcr.io/linuxserver/deluge
    container_name: deluge
    environment:
      - PUID=1001
      - PGID=115
      - TZ=Europe/Madrid
    volumes:
      - /home/pi/volumes/deluge:/config
      - /data/torrents:/data/torrents
    ports:
      - 8112:8112
      - 6881:6881
      - 6881:6881/udp
    restart: unless-stopped
```

Se puede usar `docker-compose up -d` o usar el contenido del fichero en Portainer.

# Configuración

Una vez en marcha, se puede acceder a Deluge a través del puerto `8112` (en este ejemplo [http://192.168.1.180:8112](http://192.168.1.180:8112)) y comenzar la configuración.

Lo primero y más importante es cambiar la contraseña de acceso (por defecto es `deluge`) conectándose al cliente local mediante el **Connection Manager**:

* Interface > WebUI Password > Old > `deluge`
* Interface > WebUI Password > New > `**********`
* Interface > WebUI Password > Confirm > `**********`

A continuación se configura el resto de opciones:

* Downloads > Download to > `/data/torrents/downloading`
* Downloads > Move completed to > `/data/torrents/downloaded`
* Downloads > Pre-allocate disk space
* Downloads > Network > Random port > Disable
* Plugins > Label > Enable

En algunos tutoriales se recomienda crear etiquetas para mover a un directorio concreto las descargas realizadas con **Radarr**, **Sonarr**, **Lidarr**, etc.:

* Radarr &rarr; `/data/torrents/radarr`
* Sonarr &rarr; `/data/torrents/sonarr`
* Lidarr &rarr; `/data/torrents/lidarr`

# Actualizar

Si ya se había instalado Deluge anteriormente, se puede actualizar de la siguiente manera:

```
docker stop deluge
docker rm deluge
docker rmi ghcr.io/linuxserver/deluge

# Únicamente si no se usa un 'stack' en Portainer
docker run -d \
  --name=deluge \
  -e PUID=1001 \
  -e PGID=115 \
  -e TZ=Europe/Madrid \
  -p 8112:8112 \
  -p 6881:6881 \
  -p 6881:6881/udp \  
  -v /home/pi/volumes/deluge:/config \
  -v /data/torrents:/data/torrents \
  --restart unless-stopped \
  ghcr.io/linuxserver/deluge
```

# Soporte

Algunos comandos para gestionar la configuración del contenedor:

```
# Acceder al shell mientras el contenedor está ejecutándose
docker exec -it deluge /bin/bash

# Monitorizar los logs del contenedor en tiempo real
docker logs -f deluge
```

# Referencias

* [VPSFacil Deluge](https://vpsfacil.es/deluge/)
* [Ultimate Raspberry Pi Server with Sonarr, Radarr, Jackett, NZBGet and Deluge on Docker](https://www.youtube.com/watch?v=oLxsSQIqOMw) @ LMDS GreenFrog
* [Les Tutos: Docker No. 9: Multimedia Automation Part1: Deluge & Jackett](https://www.youtube.com/watch?v=Zrz5yJ1Ytv4)
