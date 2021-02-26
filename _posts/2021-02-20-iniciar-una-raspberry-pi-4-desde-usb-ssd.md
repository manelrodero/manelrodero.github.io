---
layout : post
blog-width: true
title: 'Iniciar una Raspberry Pi 4 desde USB (SSD)'
date: '2021-02-20 10:11:06'
published: true
tags:
- Raspberry
author:
  display_name: Manel Rodero
---

Hasta hace relativamente poco las Raspberry Pi únicamente podían iniciarse desde una tarjeta SD. Estas tarjetas, además de un ancho de banda relativamente bajo (50 Mbps en una Raspberry Pi 4), no suelen durar mucho si las aplicaciones que instalamos realizan escrituras constantes.

Por estos motivos es mejor [iniciar desde un dispositivo USB](https://www.raspberrypi.org/documentation/hardware/raspberrypi/bootmodes/msd.md) (_pendrive_, disco SSD, etc.) aprovechando los puertos USB 3.0 que incorpora la Raspberry Pi 4.

Para poder iniciar desde USB es necesario tener un **bootloader** con fecha igual o superior al 3 de septiembre de 2020. Si nuestra Raspberry Pi 4 se fabricó antes, habrá que actualizar la EEPROM.

{: .box-note}
**Nota**: Se puede comprobar la versión que tenemos iniciando la Raspberry Pi sin ninguna tarjeta SD. Aparecerá una pantalla de diagnósticos con esta información.

![Bootloader sin soporte para inicio USB][1]

## Actualización de la EEPROM

Para actualizar la EEPROM se puede realizar el siguiente proceso:

* Descargar [Raspberry Pi Imager](https://www.raspberrypi.org/software/)
* Ejecutar el programa
* Escoger la imagen `Misc utility images > Raspberry Pi 4 EEPROM Boot Recovery`
* Seleccionar la tarjeta SD correcta
* Escribir la imagen en la tarjeta SD

A continuación se puede utilizar esta tarjeta SD para iniciar la Raspberry Pi y se actualizará la EEPROM de la misma. Si la luz de actividad `ACT` parpadea rápidamente en color verde y por la salida HDMI aparece una pantalla de color verde, significa que todo habrá ido correctamente.

![Bootloader con soporte para inicio USB][2]

## Raspberry Pi OS

Para iniciar un sistema operativo básico se puede escribir la imagen del mismo usando la misma herramienta:

* Ejecutar Raspberry Pi Imager
* Escoger la imagen `Raspberry Pi OS (other) > Raspberry Pi OS Lite (32-bit)`
* Seleccionar el dispositivo USB
* Escribir la imagen en el mismo

{: .box-note}
**Nota**: Antes de conectar el dispositivo a la Raspberry Pi es útil [activar el acceso por SSH](https://www.raspberrypi.org/documentation/remote-access/ssh/), sobretodo si se va a utilizar sin pantalla, teclado, etc.:

* Crear un fichero `ssh` vacío en la partición `boot`
* Reiniciar la Raspberry Pi con el dispositivo conectado por USB 3.0
* El sistema operativo, al detectar este fichero, habilita SSH y después lo borra

Una vez iniciada, únicamente es necesario conocer la dirección IP asignada por el DHCP y conectarse mediante SSH usando el usuario `pi` y el password `raspberry`.

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

Como se ha utilizado la imagen con la última EEPROM, ésta debería estar actualizada:

```
# Comprobar EEPROM
sudo rpi-eeprom-update
```
```
BCM2711 detected
VL805 firmware in bootloader EEPROM
Checking for updates in /lib/firmware/raspberrypi/bootloader/default
Use raspi-config to select either the default-production release or latest update.
BOOTLOADER: up-to-date
CURRENT: Thu  3 Sep 12:11:43 UTC 2020 (1599135103)
 LATEST: Thu  3 Sep 12:11:43 UTC 2020 (1599135103)
RELEASE: default
VL805: up-to-date
CURRENT: 000138a1
 LATEST: 000138a1
```

Mediante el comando `vcgencmd bootloader_config` se puede comprobar si el valor de [`BOOT_ORDER`](https://www.raspberrypi.org/documentation/hardware/raspberrypi/bcm2711_bootloader_config.md) es `0xf41`.

Este valor indica que se intentará iniciar por USB de forma contínua si no hay una tarjeta SD insertada en la Raspberry Pi:

```
0x1	SD CARD	SD card (or eMMC on Compute Module 4)
0x4	USB-MSD	USB mass storage boot (since 2020-09-03)
0xf	RESTART	Start again with the first boot order field. (since 2020-09-03)
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
echo "GPU => $(/opt/vc/bin/vcgencmd measure_temp)"
echo "CPU => $((cpu/1000))'C"
```

En mi caso, después de varias horas de uso, la caja está caliente sin llegar a quemar y la temperatura es la siguiente:

```
Thu 25 Feb 19:51:00 CET 2021 @ pi4nas
-------------------------------------------
GPU => temp=41.3'C
CPU => 40'C
```

Se puede probar a realizar un [_stress_](https://core-electronics.com.au/tutorials/stress-testing-your-raspberry-pi.html) de la CPU usando [`cpuburn-a7.S`](https://github.com/ssvb/cpuburn-arm/blob/master/cpuburn-a7.S) mediante las siguientes herramientas:

```
sudo apt-get install stress
wget https://github.com/ssvb/cpuburn-arm/raw/master/cpuburn-a7.S
gcc -o cpuburn-a7 cpuburn-a7.S
```

El comando a lanzar sería el siguiente:

```
while true; do vcgencmd measure_clock arm; vcgencmd measure_temp; sleep 10; done& stress -c 4 -t 900s
```
<p></p>

[1]: /assets/img/blog/2021-02-20_image_1.png "Bootloader sin soporte para inicio USB"
[2]: /assets/img/blog/2021-02-20_image_2.png "Bootloader con soporte para inicio USB"
