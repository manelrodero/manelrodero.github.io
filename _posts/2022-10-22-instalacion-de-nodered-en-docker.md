---
layout : post
blog-width: true
title: 'Instalación de Node-RED en Docker'
date: '2022-10-22 20:10:47'
published: true
tags:
- Home Assistant
- Docker
author:
  display_name: Manel Rodero
---

[Node-RED](https://nodered.org/) es una herramienta de programación para conectar dispositivos de hardware, API y servicios en línea de formas nuevas e interesantes.

Proporciona un editor basado en navegador que facilita la conexión de flujos mediante la amplia gama de nodos de la paleta que se pueden ejecutar con un solo clic.

![Imagen][3]

La [instalación en Docker](https://hub.docker.com/r/nodered/node-red) se realiza usando la imagen `nodered/node-red`.

La forma más sencilla es usar un fichero [`docker-compose.yml`](https://nodered.org/docs/getting-started/docker) con el siguiente contenido:

```
  node-red:
    image: nodered/node-red:latest
    container_name: node-red
    environment:
      - TZ=Europe/Madrid
    volumes:
      - /home/pi/volumes/node-red:/data
    ports:
      - 1880:1880
    restart: unless-stopped
```

Se puede usar `docker-compose up -d` o usar el contenido del fichero en Portainer.

# Configuración

Una vez en marcha, se puede acceder a Node-RED a través del puerto `1880` (en este ejemplo [http://192.168.1.180:1880](http://192.168.1.180:1880)) y comenzar la configuración.

# Actualizar

Si ya se había instalado Node-RED anteriormente, se puede actualizar de la siguiente manera:

```
docker stop node-red
docker rm node-red
docker rmi nodered/node-red

# Únicamente si no se usa un 'stack' en Portainer
docker run -d \
  --name=node-red \
  -e TZ=Europe/Madrid \
  -v /home/pi/volumes/node-red:/data \
  --restart unless-stopped \
  nodered/node-red:latest
```

# Soporte

Algunos comandos para gestionar la configuración del contenedor:

```
# Acceder al shell mientras el contenedor está ejecutándose
docker exec -it node-red /bin/bash

# Monitorizar los logs del contenedor en tiempo real
docker logs -f node-red
```

# Referencias

* [Instalar Home Assistant y Node-RED con Docker](https://youtu.be/wi2b5ZcySuc) @ Un loco y su tecnología
* [HA y Node-RED: Configuración rápida](https://youtu.be/ZbyT0EFzSTE) @ Un loco y su tecnología

[3]: /assets/img/blog/2022-10-22_image_3.png "Node-RED"
