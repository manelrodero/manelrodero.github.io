---
layout : post
blog-width: true
title: 'Instalación de Prowlarr en Docker'
date: '2022-05-21 09:16:48'
published: true
tags:
- Docker
author:
  display_name: Manel Rodero
---

[Prowlarr](https://github.com/Prowlarr/Prowlarr/) es una aplicación que funciona como un **servidor proxy** entre las aplicaciones ([Sonarr](https://sonarr.tv/), [Radarr](https://radarr.video/), [Lidarr](https://lidarr.audio/), [Readarr](https://readarr.com/) y [Mylar3](https://github.com/mylar3/mylar3)) y los _trackers_ Torrent o los _indexer_ Usenet.

Traduce las consultas de estas aplicaciones en consultas HTTP específicas para cada sitio de seguimiento, analiza la respuesta HTML y luego envía los resultados al software solicitante.

La [instalación en Docker](https://hub.docker.com/r/linuxserver/prowlarr) se realiza usando la imagen `linuxserver/prowlarr`.

La forma más sencilla es usar un fichero `docker-compose.yml` con el siguiente contenido:

```
  prowlarr:
    image: ghcr.io/linuxserver/prowlarr:develop
    container_name: prowlarr
    environment:
      - PUID=1001
      - PGID=115
      - TZ=Europe/Madrid
    volumes:
      - /home/pi/volumes/prowlarr:/config
    ports:
      - 9696:9696
    restart: unless-stopped
```

Se puede usar `docker-compose up -d` o usar el contenido del fichero en Portainer.

# Configuración

Una vez en marcha, se puede acceder a Radarr a través del puerto `9696` (en este ejemplo [http://192.168.1.180:9696](http://192.168.1.180:9696)) y comenzar la configuración.

Lo primero y más importante es activar el login y crear una contraseña segura para el usuario administrador:

* Settings > General > Authentication > Forms (Login page)
* Settings > General > Username > `admin`
* Settings > General > Password > `*********`
* Settings > General > Analytics > Disable Send Anonymous Usage Data

A continuación se graban los cambios y se reinicia la aplicación cuando ésta lo indique.

Para que la aplicación pueda descargar ficheros, se debe configurar un cliente Torrent o Usenet:

* Settings > Download clients > Add
  * Torrent > Deluge
  * Name: Deluge
  * Host: 192.168.1.180
  * Port: 8112
  * Password: `**********`
  * Category: prowlarr
  * Test > Save

A continuación se puede añadir uno o más **Indexer** (públicos o privados) pulsado el botón `+Add indexer` en la sección `Indexers` del menú lateral.

Estos _indexer_ se pueden añadir a Sonarr, Radarr, Lidarr, etc. de forma automática realizando los pasos siguientes:

* Settings > Applications > Add
  * Name: Radarr
  * Sync Level: Add and Remove Only
  * Prowlarr Server: `http://192.168.1.180:9696`
  * Radarr Server: `http://192.168.1.180:7878`
  * ApiKey (copiarla desde Settings > General en Radarr)
  * Test > Save

# Actualizar

Si ya se había instalado Radarr anteriormente, se puede actualizar de la siguiente manera:

```
docker stop prowlarr
docker rm prowlarr
docker rmi ghcr.io/linuxserver/prowlarr:develop

# Únicamente si no se usa un 'stack' en Portainer
docker run -d \
  --name=prowlarr \
  -e PUID=1001 \
  -e PGID=115 \
  -e TZ=Europe/Madrid \
  -p 9696:9696 \
  -v /home/pi/volumes/prowlarr:/config \
  --restart unless-stopped \
  ghcr.io/linuxserver/prowlarr:develop
```

# Soporte

Algunos comandos para gestionar la configuración del contenedor:

```
# Acceder al shell mientras el contenedor está ejecutándose
docker exec -it prowlarr /bin/bash

# Monitorizar los logs del contenedor en tiempo real
docker logs -f prowlarr
```

# Referencias

* [WikiArr: Prowlarr](https://wiki.servarr.com/en/prowlarr)
