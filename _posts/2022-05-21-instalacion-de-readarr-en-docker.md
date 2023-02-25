---
layout : post
blog-width: true
title: 'Instalación de Readarr en Docker'
date: '2022-05-21 16:17:57'
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

[Readarr](https://readarr.com/) es un administrador de colecciones de libros electrónicos para usuarios de Usenet y BitTorrent. Puede monitorizar múltiples feeds RSS para encontrar nuevos libros, descargarlos, clasificarlos y cambiarles el nombre de forma automática.

La [instalación en Docker](https://hub.docker.com/r/linuxserver/readarr) se realiza usando la imagen `linuxserver/readarr`.

La forma más sencilla es usar un fichero `docker-compose.yml` con el siguiente contenido:

```yaml
version: '3'
services:
  readarr:
    image: lscr.io/linuxserver/readarr:develop
    container_name: readarr
    environment:
      - PUID=1001
      - PGID=115
      - TZ=Europe/Madrid
    volumes:
      - /home/pi/volumes/readarr:/config
      - /data/media/books:/data/books
      - /data/torrents:/data/torrents
    ports:
      - 8787:8787
    restart: unless-stopped
```

A continuación, se puede ejecutar el comando `docker-compose up -d` o usar el contenido del fichero en Portainer.

# Configuración

Una vez en marcha, se puede acceder a Readarr a través del puerto `8787` (en este ejemplo [http://192.168.1.180:8787](http://192.168.1.180:8787)) y comenzar la configuración.

Lo primero y más importante es activar el login y crear una contraseña segura para el usuario administrador:

* Settings > General > Authentication > Forms (Login page)
* Settings > General > Username > `admin`
* Settings > General > Password > `*********`
* Settings > General > Analytics > Disable Send Anonymous Usage Data

A continuación se graban los cambios y se reinicia la aplicación cuando ésta lo indique.

Para que la aplicación pueda descargar ficheros, se debe configurar un cliente Bittorrent o Usenet:

* Settings > Download clients > Add
  * Torrent > Deluge
  * Name: Deluge
  * Host: 192.168.1.180
  * Port: 8112
  * Password: `**********`
  * Category: readarr
  * Test > Save

Para encontrar los libros hay que añadir uno o más **indexer** de Jackett haciendo lo siguiente:

* Settings > Indexers > Add > Torznab > Custom
  * Name: Jackett
  * URL &rarr; Copiarla desde Jackett
  * API Key &rarr; Copiarla desde Jackett
  * Ajustar las categorías

# Actualizar

Si ya se había instalado Readarr anteriormente, se puede actualizar de la siguiente manera:

```
docker stop readarr
docker rm readarr
docker rmi lscr.io/linuxserver/readarr:develop
docker-compose up -d
```

# Soporte

Algunos comandos para gestionar la configuración del contenedor:

```
# Acceder al shell mientras el contenedor está ejecutándose
docker exec -it readarr /bin/bash

# Monitorizar los logs del contenedor en tiempo real
docker logs -f readarr
```

# Referencias

* [WikiArr: Readarr](https://wiki.servarr.com/en/readarr)
