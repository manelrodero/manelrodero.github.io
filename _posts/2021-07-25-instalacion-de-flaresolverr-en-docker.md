---
layout : post
blog-width: true
title: 'Instalación de FlareSolverr en Docker'
date: '2021-07-25 17:19:11'
published: true
tags:
- Docker
author:
  display_name: Manel Rodero
---

#### _**Actualizaciones**:_

* **2023-01-15**: Revisión del documento y corrección de errores.

# Instalación

[FlareSolverr](https://github.com/FlareSolverr/FlareSolverr) es un servidor proxy para saltarse las protecciones Cloudflare y DDoS-GUARD.

La [instalación en Docker](https://hub.docker.com/r/flaresolverr/flaresolverr) se realiza usando la imagen `flaresolverr/flaresolverr`.

La forma más sencilla es usar un fichero `docker-compose.yml` con el siguiente contenido:

```yaml
version: '3'
services:
  flaresolverr:
    image: ghcr.io/flaresolverr/flaresolverr:latest
    container_name: flaresolverr
    environment:
      - LOG_LEVEL=${LOG_LEVEL:-info}
      - LOG_HTML=${LOG_HTML:-false}
      - CAPTCHA_SOLVER=${CAPTCHA_SOLVER:-none}
      - TZ=Europe/Madrid
    ports:
      - "${PORT:-8191}:8191"
    restart: unless-stopped
```

Se puede usar `docker-compose up -d` o usar el contenido del fichero en Portainer.

# Configuración

Una vez en marcha, se puede acceder a FlareSolverr a través del puerto `8191` (en este ejemplo [http://192.168.1.180:8191](http://192.168.1.180:8191)) para comprobar que está funcionando correctamente:

```json
{
  "msg": "FlareSolverr is ready!",
  "version": "3.0.2",
  "userAgent": "Mozilla/5.0 (X11; Linux aarch64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36"
}
```

# Actualizar

Si ya se había instalado FlareSolverr anteriormente, se puede actualizar de la siguiente manera:

```
docker stop flaresolverr
docker rm flaresolverr
docker rmi ghcr.io/linuxserver/flaresolverr
docker-compose up -d
```

# Soporte

Algunos comandos para gestionar la configuración del contenedor:

```
# Acceder al shell mientras el contenedor está ejecutándose
docker exec -it flaresolverr /bin/bash

# Monitorizar los logs del contenedor en tiempo real
docker logs -f flaresolverr
```

# Referencias

* [Configuring FlareSolverr]( https://github.com/Jackett/Jackett#configuring-flaresolverr)
