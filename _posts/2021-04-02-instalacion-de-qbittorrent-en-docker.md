---
layout : post
blog-width: true
title: 'Instalación de qBittorrent en Docker'
date: '2021-04-02 20:48:35'
published: true
tags:
- Docker
author:
  display_name: Manel Rodero
---

[qBittorrent](https://www.qbittorrent.org/) es una alternativa de código abierto a uTorrent.

La [instalación en Docker](https://hub.docker.com/r/linuxserver/qbittorrent) se realiza usando la imagen `linuxserver/qbittorrent`.

La forma más sencilla es usar un fichero `docker-compose.yml` con el siguiente contenido:

```
  qbittorrent:
    image: ghcr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    environment:
      - PUID=1001
      - PGID=115
      - TZ=Europe/Madrid
      - WEBUI_PORT=8080
    volumes:
      - /home/pi/volumes/qbittorrent:/config
      - /data/torrents:/data/torrents
    ports:
      - 8080:8080
      - 6882:6882
      - 6882:6882/udp
    restart: unless-stopped
```

Se puede usar `docker-compose up -d` o usar el contenido del fichero en Portainer.

# Configuración

Una vez en marcha, se puede acceder a qBittorrent a través del puerto `8080` (en este ejemplo [http://192.168.1.180:8080](http://192.168.1.180:8080)) y comenzar la configuración.

Lo primero y más importante es cambiar la contraseña de acceso del usuario `admin` (por defecto es `adminadmin`):

* Tools > Options > Web UI > Authentication > Password > `**********`

A continuación se configura el resto de opciones:

* Tools > Options
  * Downloads > Append .!qB extension to incomplete files
  * Default Save Path > `/data/torrents/downloaded`
  * Keep incomplete torrents in > `/data/torrents/downloading`

# Troubleshooting

No se puede acceder usando la contraseña por defecto, no se carga la WebUI de qBittorrent y se queda en la pantalla de _login_.

Después de haberlo dado por imposible, la solución para un problema similar en **Radarr** (actualizar `libseccomp2`) también ha servido para acceder a la interfaz web.
