---
layout : post
blog-width: true
title: 'Instalar Raspberry Pi OS 64-bits'
date: '2021-07-18 19:38:15'
published: true
tags:
- HomeLab
- Raspberry
author:
  display_name: Manel Rodero
---

Hace unos meses [instalé Raspberry Pi OS 32-bits](/blog/iniciar-una-raspberry-pi-4-desde-usb-ssd) en una Raspberry Pi 4 iniciando desde SSD para utilizarla como _Home Server_ con Samba, Docker, etc.

Ahora toca reinstalarla utilizando **Raspberry Pi OS 64-bits** (es una versión Beta pero parece que es bastante estable) para obtener un mayor aprovechamiento de la arquitectura y memoria del dispositivo.

## Raspberry Pi OS Lite 64-bits

Para poder instalar esta versión con la herramienta [_Raspberry Pi Imager_](https://www.raspberrypi.org/software/) es necesario descargar manualmente la última versión de la imagen disponible a día de hoy (**2021-05-07**).

* Descargar el fichero [`2021-05-07-raspios-buster-arm64-lite.zip`](https://downloads.raspberrypi.org/raspios_lite_arm64/images/raspios_lite_arm64-2021-05-28/2021-05-07-raspios-buster-arm64-lite.zip)
* Ejecutar Raspberry Pi Imager
* Escoger la opción `Use custom` y seleccionar el fichero anterior
* Seleccionar el dispositivo USB
* Escribir la imagen en el mismo

{: .box-note}
**Nota**: Antes de conectar el dispositivo a la Raspberry Pi es útil [activar el acceso por SSH](https://www.raspberrypi.org/documentation/remote-access/ssh/), sobretodo si se va a utilizar sin pantalla, teclado, etc.:

* Crear un fichero `ssh` vacío en la partición `boot`
* Reiniciar la Raspberry Pi con el dispositivo conectado por USB 3.0
* El sistema operativo, al detectar este fichero, habilita SSH y después lo borra

Una vez iniciada, únicamente es necesario conocer la dirección IP asignada por el DHCP y conectarse mediante SSH usando el usuario `pi` y el password `raspberry`.

## Comprobación arquitectura

Se puede comprobar la arquitectura de Raspberry Pi OS y la versión del compilador **GNU C++** (necesario para compilar algunos programas) usando los comandos:

```
# Debería indicar aarch64
uname -a
```
```
Linux raspberrypi 5.10.17-v8+ #1421 SMP PREEMPT Thu May 27 14:01:37 BST 2021 aarch64 GNU/Linux
```

```
# Debería indicar --build=aarch64-linux-gnu
gcc -v
```

## Actualización de Raspberry Pi OS

Para [actualizar Raspberry Pi OS](https://www.raspberrypi.org/documentation/raspbian/updating.md) a la última versión se tienen que ejecutar los siguientes comandos:

```
# Actualizar la lista de repositorios
sudo apt update
# Actualizar los paquetes instalados a la última versión
sudo apt upgrade -y
# Reiniciar
sudo reboot
```

{: .box-note}
**Nota**: Los ingenieros de Raspberry [desaconsejan utilizar `rpi-update`](https://www.raspberrypi.org/documentation/raspbian/applications/rpi-update.md) ya que instala versiones que aún no son estables.

A continuación es interesante comprobar si hay actualizaciones de la **EEPROM** que pueden ayudar con el rendimiento y la disipación del calor:

```
# Comprobar EEPROM
sudo rpi-eeprom-update
```
```
*** UPDATE AVAILABLE ***
BOOTLOADER: update available
   CURRENT: Thu  3 Sep 12:11:43 UTC 2020 (1599135103)
    LATEST: Thu 29 Apr 16:11:25 UTC 2021 (1619712685)
   RELEASE: default (/lib/firmware/raspberrypi/bootloader/default)
            Use raspi-config to change the release.

  VL805_FW: Using bootloader EEPROM
     VL805: up to date
   CURRENT: 000138a1
    LATEST: 000138a1
```

```
# Actualizar la EEPROM (en el caso que sea necesario)
sudo rpi-eeprom-update -a
sudo reboot
```

## Configuración

Se puede cambiar éste y otros valores usando el comando [`sudo raspi-config`](https://www.raspberrypi.org/documentation/configuration/raspi-config.md):

* System Options
  * Password: raspberry &rarr; nuevo password
  * Hostname: raspberry &rarr; pi4nas
* Localisation Options
  * Timezone: Europe/London &rarr; Europe/Madrid
  * Keyboard
  * WLAN Country: ES

## Temperatura

Al estar utilizando disipación pasiva con una caja de aluminio [Geekwork TB-2020-05-1](https://amzn.to/3kbIeIL), es recomendable **comprobar la temperatura** usando el siguiente [script](https://www.cyberciti.biz/faq/linux-find-out-raspberry-pi-gpu-and-arm-cpu-temperature-command/):

```
#!/bin/bash
# Script: my-pi-temp.sh
# Purpose: Display the ARM CPU and GPU  temperature of Raspberry Pi 2/3 
# Author: Vivek Gite <www.cyberciti.biz> under GPL v2.x+
# -------------------------------------------------------
cpu=$(</sys/class/thermal/thermal_zone0/temp)
echo "$(date) @ $(hostname)"
echo "-------------------------------------------"
echo "GPU => $(vcgencmd measure_temp)"
echo "CPU => temp=$((cpu/1000))'C"
```

En mi caso, después de varias horas de uso, la caja está caliente sin llegar a quemar y la temperatura es la siguiente:

```
Sun 18 Jul 21:26:01 CEST 2021 @ pi4nas
-------------------------------------------
GPU => temp=49.1'C
CPU => temp=48'C
```
<p></p>
