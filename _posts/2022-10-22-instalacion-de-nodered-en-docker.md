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

#### _**Actualizaciones**:_

* **2023-03-01**: Revisión del documento y corrección de errores.
* **2023-03-02**: Integración con HA

# Introducción

[Node-RED](https://nodered.org/) es una herramienta de programación para conectar dispositivos de hardware, API y servicios en línea de formas nuevas e interesantes.

Proporciona un editor basado en navegador que facilita la conexión de flujos mediante la amplia gama de nodos de la paleta que se pueden ejecutar con un solo clic.

![Node-RED][3]

# Instalación

La [instalación en Docker](https://hub.docker.com/r/nodered/node-red) se realiza usando la imagen `nodered/node-red`.

La forma más sencilla es usar un fichero [`docker-compose.yml`](https://nodered.org/docs/getting-started/docker) con el siguiente contenido:

```yaml
version: "3"
services:
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

Una opción recomendable, además de utilizar HTTPS, es utilizar la autenticación de usuarios.

Para ello hay que descomentar esta sección del fichero [`settings.js`](https://nodered.org/docs/user-guide/runtime/settings-file) para disponer de un usuario `admin` con todos los privilegios:

```js
    adminAuth: {
        type: "credentials",
        users: [{
            username: "admin",
            password: "$2a$08$zZWtXTja0fB1pzD4sHCMyOCMYz2Z6dNbM6tl8sJogENOMcxWV9DN.",
            permissions: "*"
        }]
    },
```

Para [generar la contraseña](https://nodered.org/docs/user-guide/runtime/securing-node-red#generating-the-password-hash) de este usuario hay que utilizar la herramienta `node-red` de la siguiente manera::

```
pi@pi4nas:~ $ docker exec -it node-red node-red admin hash-pw
Password:
$2a$08$zZWtXTja0fB1pzD4sHCMyOCMYz2Z6dNbM6tl8sJogENOMcxWV9DN.
```

A continuación se reinicia el contenedor para utilizar la nueva configuración.

# Integración con HA

La integración de Node-RED con Home Assistant se reliza utilizando [HACS](instalacion-de-hacs-en-home-assistant-docker):

* HACS &rarr; Integrations
* Pulsar el botón `Explore & Download Repositories`
* Buscar [**Node-RED Companion**](https://github.com/zachowj/hass-node-red) y seleccionarlo
* Pulsar el botón `Download`
* Elegir la versión y pulsar en `Download` de nuevo
* Reiniciar Home Assistant
* Settings &rarr; Devices & Services
* Pulsar el botón `Add integration`
* Buscar la integración `Node-RED Companion` y seleccionarla
* Pulsar el botón `Submit` para agregarla

A continuación es necesario agregar un módulo en Node-RED que permitirá comunicarse a través de _websockets_ y una API REST:

* Menú &rarr; Manage Palette
* Seleccionar la pestaña `Install`
* Buscar **node-red-contrib-home-assistant-websocket** y seleccionarlo
* Oulsar el botón `Install`

Al cabo de unos segundos se habrán añadido 27 nuevos **nodos** a la **paleta** de Node-RED:

![Paleta HA Websockets][4]

# Actualizar

Si ya se había instalado Node-RED anteriormente, se puede actualizar de la siguiente manera:

```
docker stop node-red
docker rm node-red
docker rmi nodered/node-red
docker-compose up -d
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
* [HA y Node-RED: Configuración rápida](https://www.youtube.com/watch?v=sIvzWJ4ytok) @ Un loco y su tecnología
* [Basics: Connecting Home-Assistant to Node-red](https://www.thesmarthomebook.com/2021/04/14/basics-connecting-home-assistant-to-node-red/)

[3]: /assets/img/blog/2022-10-22_image_3.png "Node-RED"
[4]: /assets/img/blog/2022-10-22_image_4.png "Paleta HA websockets"
