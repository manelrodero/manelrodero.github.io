---
layout : post
blog-width: true
title: 'Instalación de Jackett en Docker'
date: '2021-04-02 19:27:35'
published: true
tags:
- Docker
author:
  display_name: Manel Rodero
---

#### _**Actualizaciones**:_

* **2022-12-03**: Revisión del documento y corrección de errores.

# Instalación

[Jackett](https://github.com/Jackett/Jackett) es una aplicación que funciona como un **servidor proxy** entre las aplicaciones ([Sonarr](https://sonarr.tv/), [Radarr](https://radarr.video/), [Lidarr](https://lidarr.audio/), etc.) y los _trackers_ de ficheros Torrent.

Traduce las consultas de estas aplicaciones en consultas HTTP específicas para cada sitio de seguimiento, analiza la respuesta HTML y luego envía los resultados al software solicitante.

La [instalación en Docker](https://hub.docker.com/r/linuxserver/jackett/) se realiza usando la imagen `linuxserver/jackett`.

La forma más sencilla de hacerlo es usar un fichero `docker-compose.yml` con el siguiente contenido:

```yaml
version: '3'
services:
  jackett:
    image: lscr.io/linuxserver/jackett:latest
    container_name: jackett
    environment:
      - PUID=1001
      - PGID=115
      - TZ=Europe/Madrid
      - AUTO_UPDATE=false
    volumes:
      - ~/volumes/jackett:/config
      - /data/torrents:/data/torrents
    ports:
      - 9117:9117
    restart: unless-stopped
```

A continuación, se puede ejecutar el comando `docker-compose up -d` o usar el contenido del fichero en Portainer.

# Configuración

Una vez en marcha, se puede acceder a Jackett a través del puerto `9117` (en este ejemplo [http://192.168.1.180:9117](http://192.168.1.180:9117)) y comenzar la configuración.

Lo primero y más importante es crear una contraseña para el usuario administrador y deshabilitar las actualizaciones:

* Jackett Configuration
  * Admin password &rarr; `*********`
  * Disable auto update &rarr; true

A continuación se puede añadir uno o más **Indexer** (públicos o privados) pulsado el botón `+Add indexer` en la parte superior de la pantalla.

Estos _indexer_ se pueden añadir después a Sonarr, Radarr, Lidarr, etc. realizando los pasos siguientes:

* Copiar la URL del _indexer_ haciendo clic en el botón `Copy Torznab Feed` de Jackett
* En Sonarr, Radarr, Lidarr, etc. ir a Settings > Indexers > Add > Torznab > Custom
* Pegar la URL copiada anteriormente
* Pegar la API Key de Jackett
* Configurar los ID de las categorías

También se puede [configurar la URL de **FlareSolverr**](https://github.com/Jackett/Jackett#configuring-flaresolverr) para aquellos índices que estén protegidos por Cloudflare (en este ejemplo [http://192.168.1.180:8191/](http://192.168.1.180:8191/)).

# Actualizar

Si ya se había instalado Jackett anteriormente, se puede actualizar de la siguiente manera:

```
docker stop jackett
docker rm jackett
docker rmi lscr.io/linuxserver/jackett
docker-compose up -d
```

# Soporte

Algunos comandos para gestionar la configuración del contenedor:

```
# Acceder al shell mientras el contenedor está ejecutándose
docker exec -it jackett /bin/bash

# Monitorizar los logs del contenedor en tiempo real
docker logs -f jackett
```

# Referencias

* [VPSFacil Jackett](https://vpsfacil.es/jackett/)
* [Ultimate Media Server : Episode 3 - Configuring Jackett, Sonarr, and Radarr](https://youtu.be/uvc4TnhVecA) @ DB Tech
* [Ultimate Raspberry Pi Server with Sonarr, Radarr, Jackett, NZBGet and Deluge on Docker](https://www.youtube.com/watch?v=oLxsSQIqOMw) @ 
LMDS GreenFrog
* [Les Tutos: Docker No. 9: Multimedia Automation Part1: Deluge & Jackett](https://www.youtube.com/watch?v=Zrz5yJ1Ytv4)
