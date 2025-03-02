---
layout : post
blog-width: true
title: 'Instalación de Heimdall en Docker'
date: '2021-04-03 19:49:15'
published: true
tags:
- HomeLab
- Docker
author:
  display_name: Manel Rodero
---

[Heimdall](https://heimdall.site/){:target="_blank"} es un _dashboard_ para aplicaciones web.

La [instalación en Docker](https://hub.docker.com/r/linuxserver/heimdall/){:target="_blank"} se realiza usando la imagen `linuxserver/heimdall`.

La forma más sencilla es usar un fichero `docker-compose.yml` con el siguiente contenido:

```docker
  heimdall:
    image: ghcr.io/linuxserver/heimdall:latest
    container_name: heimdall
    environment:
      - PUID=1001
      - PGID=115
      - TZ=Europe/Madrid
    volumes:
      - /home/pi/volumes/heimdall:/config    
    ports:
      - 80:80
      - 443:443
    restart: always
```

Se puede usar `docker-compose up -d` o usar el contenido del fichero en Portainer.

# Configuración

Una vez en marcha, se puede acceder a Heimdall a través del puerto `80` (en este ejemplo [http://192.168.1.180/](http://192.168.1.180/){:target="_blank"}) y comenzar la configuración.

Lo primero y más importante es crear una contraseña para el usuario `admin` accediendo a la seccion **Users**.

A continuación se pueden añadir aplicaciones:

* Portainer &rarr; `http://192.168.1.180:9000`
* Deluge &rarr; `http://192.168.1.180:8112`
* Jackett &rarr; `http://192.168.1.180:9117`
* Prowlarr &rarr; `http://192.168.1.180:9696`
* Sonarr &rarr; `http://192.168.1.180:8989`
* Radarr &rarr; `http://192.168.1.180:7878`
* Lidarr &rarr; `http://192.168.1.180:8686`
* Bazarr &rarr; `http://192.168.1.180:6767`
* Readarr &rarr; `http://192.168.1.180:8787`

# Actualizar

Si ya se había instalado Heimdall anteriormente, se puede actualizar de la siguiente manera:

```plaintext
docker stop heimdall
docker rm heimdall
docker rmi ghcr.io/linuxserver/heimdall

# Únicamente si no se usa un 'stack' en Portainer
docker run -d \
  --name=heimdall \
  -e PUID=1001 \
  -e PGID=115 \
  -e TZ=Europe/Madrid \
  -p 80:80 \
  -p 443:443 \
  -v /home/pi/volumes/heimdall:/config \
  --restart unless-stopped \
  ghcr.io/linuxserver/heimdall:latest
```

# Soporte

Algunos comandos para gestionar la configuración del contenedor:

```plaintext
# Acceder al shell mientras el contenedor está ejecutándose
docker exec -it heimdall /bin/bash

# Monitorizar los logs del contenedor en tiempo real
docker logs -f heimdall
```

# Referencias

* [Heimdall, tu propio portal de inicio. Instalación y configuracion](https://domology.es/heimdall-instalacion-y-configuracion/){:target="_blank"}
