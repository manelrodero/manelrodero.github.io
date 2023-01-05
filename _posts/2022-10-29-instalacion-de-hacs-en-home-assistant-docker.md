---
layout : post
blog-width: true
title: 'Instalación de HACS en Home Assistant (Docker)'
date: '2022-10-29 21:46:54'
published: true
tags:
- Home Assistant
- Docker
author:
  display_name: Manel Rodero
---

La _Home Assistant Community Store_ o [HACS](https://hacs.xyz/) es una herramienta que permite descargar diferentes integraciones y temas no oficiales de una forma sencilla.

Uno de los requisitos para ejecutar HACS, además de una versión soportada de Home Assistant superior a la 2022.10.0, es tener habilitada la [integración My](https://www.home-assistant.io/integrations/my/).

La instalación se puede realizar desde el propio _host_ (una Raspberry Pi en mi caso) o accediendo al _docker_ en el que se ejecuta Home Assistant mediante el comando `docker exec -it homeassistant bash`.

* Abrir un terminal con el host (p.ej. mediante **SSH**)
* Acceder al directorio que contiene la configuración de Home Assistant:
  * Host: `/home/pi/volumes/homeassistant`
  * Docker: `/config`
* Ejecutar el _script_ que descarga HACS `wget -O - https://get.hacs.xyz | bash -`

En este ejemplo se ha realizado la instalación desde el _docker_:

```
pi@pi4nas:~ $ docker exec -it homeassistant /bin/bash
bash-5.1# cd /config
bash-5.1# wget -O - https://get.hacs.xyz | bash -
Connecting to get.hacs.xyz (172.67.143.44:443)
Connecting to raw.githubusercontent.com (185.199.108.133:443)
writing to stdout
-                    100% |************************************************************************|  2742  0:00:00 ETA
written to stdout
INFO: Trying to find the correct directory...
INFO: Found Home Assistant configuration directory at '/config'
INFO: Creating custom_components directory...
INFO: Changing to the custom_components directory...
INFO: Downloading HACS
Connecting to github.com (140.82.121.4:443)
Connecting to github.com (140.82.121.4:443)
Connecting to objects.githubusercontent.com (185.199.111.133:443)
saving to 'hacs.zip'
hacs.zip             100% |************************************************************************| 3840k  0:00:00 ETA
'hacs.zip' saved
INFO: Creating HACS directory...
INFO: Unpacking HACS...
INFO: Removing HACS zip file...
INFO: Installation complete.

INFO: Remember to restart Home Assistant before you configure it
```

A continuación se reinicia el contenedor de Home Assistant y se vuelve a iniciar sesión para proceder a la **configuración inicial** de HACS:

* Ir a `Settings > Devices & Services`
* Seleccionar `Add integration`
* Buscar `HACS` y seleccionarla para instalar
* Aceptar las siguientes indicaciones:
  * [x] I know how to access Home Assistant logs
  * [x] I know that there are no add-ons in HACS
  * [x] I know that everything inside HACS is custom and untested by Home Assistant
  * [x] I know that if I get issues with Home Assistant I should disable all my custom_components
* Activar el dispositivo:
  * Abrir la página `https://github.com/login/device`
  * Introducir el código proporcionado por Home Assistant
  * [Autorizar](https://hacs.xyz/docs/faq/github_account/) a HACS en GitHub ([OAuth](https://docs.github.com/en/developers/apps/building-oauth-apps/authorizing-oauth-apps))
* Reiniciar Home Assistant

![HACS OAuth][1]

> Si en algún momento se [elimina HACS de Home Assistant](https://hacs.xyz/docs/setup/remove/), hay que acordarse de [borrar la autorización en GitHub](https://github.com/settings/applications).

Ahora, con HACS instalado, ya se pueden instalar algunas **tarjetas** interesantes:

* [PVPC Hourly Pricing Card](https://github.com/danimart1991/pvpc-hourly-pricing-card) de [@danimart1991](https://twitter.com/danimart1991)
* [Home Assistant Sun Card](https://github.com/AitorDB/home-assistant-sun-card) de [@Aitor_DB](https://twitter.com/Aitor_DB)

[1]: /assets/img/blog/2022-10-29_image_1.png "HACS OAuth"
