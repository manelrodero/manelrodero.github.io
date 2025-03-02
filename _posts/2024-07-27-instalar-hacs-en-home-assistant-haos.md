---
layout : post
blog-width: true
title: 'Instalar HACS en Home Assistant (HAOS)'
date: '2024-07-27 13:31:08'
#last-updated: '2024-07-27 13:31:08'
published: true
tags:
- Home Assistant
author:
  display_name: Manel Rodero
#cover-img: "/assets/img/blog/2024-07-27_cover.png"
thumbnail-img: ""
---

Para seguir el proceso de migración de mi _Home Lab_ desde una pequeña [Raspberry Pi 4](instalar-raspberry-pi-os-64bits) a un [OptiPlex 7050 ejecutando Proxmox](proxmox-ve-802-en-un-dell-optiplex-7050) ahora le toca el turno a [**HACS**](https://hacs.xyz/){:target="_blank"}, un repositorio no oficial de integraciones, _add-ons_, etc. mantenido por la comunidad de Home Assistant.

En esta ocasion, en lugar de realizar la [instalación de HACS en Home Assistant (Docker)](instalacion-de-hacs-en-home-assistant-docker), se instalará en la propia VM de Home Assistant OS en Proxmox.

# Introducción

Para poder instalar HACS es necesario acceder a la _shell_ de Home Assistant OS. Esto se puede realizar accediendo a la _Console_ de la VM de HAOS en Proxmox y en el CLI de Home Assistant escribir el comando `login`.

{: .box-note}
La letra del CLI es bastante pequeña y el teclado está configurado en `us-US` por lo que será más cómodo realizar esta instalación usando un **Terminal** desde la propia interfaz web de Home Assistant OS.

Para ello hay que habilitar el modo avanzado:

* Acceder al _profile_ del usuario `admin`
* User settings &rarr; Advanced mode &rarr; **Enable**

Y después instalar un terminal SSH:

* Settings &rarr; Add-ons &rarr; Add-on Store
* Buscar e instalar `Terminal & SSH`
* Habilitar las siguientes opciones:
  * Watchdog
  * Auto update
  * Show in sidebar
* Pulsar en `START`

Si todo funciona correctamente, en el menú lateral habrá aparecido la opción `Terminal` que podemos pulsar para acceder a un terminal SSH dentro de Home Assistant OS (con la letra más grande y el teclado `es-ES` correctamente configurado).

A partir de aquí la instalación y configuración de HACS se realiza de la misma manera que en la [instalación de HACS en Home Assistant (Docker)](instalacion-de-hacs-en-home-assistant-docker):

```bash
wget -nv -O hacs.sh https://get.hacs.xyz
# Revisar el script ;-)
bash hacs.sh
```

A continuación se reinicia Home Asistant y se añade la nueva integración HACS instalada anteriormente:

* Ir a `Settings > System`
* Seleccionar `Restart`
* Ir a `Settings > Devices & Services`
* Seleccionar `Add integration`
* Buscar `HACS` y seleccionarla para instalar
* Aceptar las siguientes indicaciones:
  * [x] I know how to access Home Assistant logs
  * [x] I know that there are no add-ons in HACS
  * [x] I know that everything inside HACS including HACS itself is custom and untested by Home Assistant
  * [x] I know that if I get issues with Home Assistant I should disable all my custom_components
  * [x] Enable experimental features, this is what eventually will become HACS 2.0.0, if you enable it now you do not need to do anything when 2.0.0 is released
* Activar el dispositivo:
  * Abrir la página `https://github.com/login/device`
  * Introducir el código proporcionado por Home Assistant
  * [Autorizar](https://hacs.xyz/docs/faq/github_account/) a HACS en GitHub ([OAuth](https://docs.github.com/en/developers/apps/building-oauth-apps/authorizing-oauth-apps))
* Reiniciar Home Assistant

Ahora, con HACS instalado, ya se pueden instalar algunas **tarjetas Lovelace** interesantes:

* [PVPC Hourly Pricing Card](https://github.com/danimart1991/pvpc-hourly-pricing-card){:target="_blank"}
* [Lovelace Horizon Card](https://github.com/rejuvenate/lovelace-horizon-card){:target="_blank"}

### Historial de cambios

* **2024-07-27**: Documento inicial
