---
layout : post
blog-width: true
title: 'Instalación de Home Assistant en Docker'
date: '2022-10-22 19:02:51'
published: true
tags:
- Home Assistant
- Docker
author:
  display_name: Manel Rodero
---

[Home Assistant](https://www.home-assistant.io/) es un software que permite **automatizar** y **controlar** la **domótica** de un hogar.

Las principales ventajas respecto a otras soluciones existentes para controlar la domótica del hogar son:

* Es un software gratuito de código abierto
* Es un software que se ejecuta **localmente**, es decir, dentro de la infraestructura creada en casa
* Soporta diferentes sistemas de comunicación (Wi-Fi, Bluetooth, etc.)
* Dispone de miles de **integraciones** con todo tipo de dispositivos útiles para el hogar (luces inteligentes, termostatos, sensores, etc.)
* Soporta diferentes protocolos de domótica (ZigBee, Z-Wave, Amazon Alexa, etc.)

> [Alexa](https://alexa.amazon.com), por ejemplo, funciona a través de los servidores de Amazon y requiere de conexión a Internet.

![Home Assistant][1]

Home Assistant se puede instalar en diferentes [plataformas](https://www.home-assistant.io/installation/) y usando diferentes métodos:

* **Home Assistant Operating System**: mínimo sistema operativo optimizado para Home Assistant. Incluye el Supervisor para instalar _addons_ y hacer copias de seguridad.
* **Home Assistant Container**: instalación básica de Home Assistant Core basada en contenedores (p.ej. Docker)
* **Home Assistant Supervised**: instalación manual del Supervisor
* **Home Assistant Core**: instalación manual usando un entorno virtual de Python

La [instalación en Docker](https://hub.docker.com/r/linuxserver/homeassistant) se realiza usando la imagen `linuxserver/homeassistant`.

La forma más sencilla es usar un fichero [`docker-compose.yml`](https://www.home-assistant.io/installation/raspberrypi#docker-compose) con el siguiente contenido:

```
version: '3'
services:
  homeassistant:
    image: ghcr.io/home-assistant/home-assistant:stable
    container_name: homeassistant
    environment:
      - TZ=Europe/Madrid
    volumes:
      - /home/pi/volumes/homeassistant:/config
      - /etc/localtime:/etc/localtime:ro
    restart: unless-stopped
    privileged: true
    network_mode: host
```

Se puede usar `docker-compose up -d` o usar el contenido del fichero en Portainer.

# Configuración

Una vez en marcha, se puede acceder a Home Assistant a través del puerto `8123` (en este ejemplo [http://192.168.1.180:8123](http://192.168.1.180:8123)) y comenzar la configuración.

Lo primero y más importante es crear un usuario administrador desde el asistente inicial:

* Name > `admin`
* Username > `admin`
* Password > `*********`
* Confirm password > `*********`

A continuación se graban los cambios y se continúa con el asistente para indicar el nombre de la instalación:

* Name of your Home Assistant installation > `Casa`

Algunas de las siguientes opciones no hace falta configurarlas en este momento por lo que se puede pulsar el botón `Next`:

* Where you live: `Amsterdam` (por defecto)
* Time Zone
* Elevation: `0` (Metros)
* Unit System: `Metric` (Celsius, Kilogramos)
* Currency: `EUR`

En la pantalla para compartir información sobre uso y estadísticas de forma anónima también pulsamos sobre el botón `Next`:

* Basic analytics: `Off`
* Usage: `Off`
* Statistical data: `Off`
* Diagnostics: `Off`

En la pantalla final del asistente se indican los dispositivos que se hayan encontrado en la red pero no hace falta configurarlos ahora y pulsamos sobre el botón `Finish`.

![Dispositivos en Home Assistant][2]

# ¿Y ahora qué?

Ahora se abre un mundo lleno de posibilidades que debería comenzar por leer la propia [documentación](https://www.home-assistant.io/docs/) del proyecto para aprender a utilizar los [_scripts_](https://www.home-assistant.io/docs/scripts/) y crear nuestros propios [tableros](https://www.home-assistant.io/dashboards/) a partir de las miles de [integraciones](https://www.home-assistant.io/integrations/) existentes.

El [blog](https://www.home-assistant.io/blog/) del proyecto y la página con [ejemplos de uso](https://www.home-assistant.io/examples/) de la comunidad son una fuente de inspiración para aprender a sacar todo el partido a nuestra instalación de Home Assistant.

Yo, de momento, he [integrado NUT en Home Assistant](integrar-nut-en-home-assistant) para [monitorizar el SAI](monitorizar-un-sai-ups-desde-una-raspberry-pi) que protege la informática de casa. También he [instalado Node-RED](instalacion-de-nodered-en-docker) para realizar futuras automatizaciones de forma más visual.

Ahora, ¡a seguir explorando!

# Actualizar

Si ya se había instalado Home Assistant anteriormente, se puede actualizar de la siguiente manera:

```
docker stop homeassistant
docker rm homeassistant
docker rmi ghcr.io/home-assistant/home-assistant

# Únicamente si no se usa un 'stack' en Portainer
docker run -d \
  --name=homeassistant \
  --privileged \
  -e TZ=Europe/Madrid \
  -v /home/pi/volumes/homeassistant:/config \
  -v /etc/localtime:/etc/localtime:ro \
  --restart unless-stopped \
  --network=host \
  ghcr.io/home-assistant/home-assistant:stable
```

# Soporte

Algunos comandos para gestionar la configuración del contenedor:

```
# Acceder al shell mientras el contenedor está ejecutándose
docker exec -it homeassistant /bin/bash

# Monitorizar los logs del contenedor en tiempo real
docker logs -f homeassistant
```

# Referencias

* [Instalar Home Assistant y Node-RED con Docker](https://youtu.be/wi2b5ZcySuc) @ Un loco y su tecnología
* [Living without add-ons on Home Assistant Container](https://www.youtube.com/watch?v=DV_OD4OPKno) @ Home Automation Guy
* [Home Assistant Community Store (HACS)](https://hacs.xyz/), la tienda de integraciones, _plugins_, etc. de la comunidad

[1]: /assets/img/blog/2022-10-22_image_1.png "Home Assistant"
[2]: /assets/img/blog/2022-10-22_image_2.png "Dispositivos en HA"
