---
layout : post
blog-width: true
title: 'Temperatura y humedad en Home Assistant con Xiaomi LYWSD03MMC'
date: '2023-06-22 20:09:51'
last-updated: '2025-01-12 13:55:27'
published: true
tags:
- Home Assistant
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2023-06-22_cover.png"
thumbnail-img: ""
---

El [**Xiaomi LYWSD03MMC**](https://ams.buy.mi.com/es/item/3204600074){:target="_blank"} es un pequeño monitor de temperatura y humedad con display que se vende a unos precios muy asequibles:

* [Tienda oficial Mi.com](https://ams.buy.mi.com/es/item/3204600074?skupanel=1){:target="_blank"}, 9,99€
* [Tienda Mi Fans Store](https://es.aliexpress.com/item/1005006605707689.html){:target="_blank"}, 6,09€
* [Tienda Xiaomi Smart-Technology Store](https://es.aliexpress.com/item/1005007283233608.html){:target="_blank"}, 5,99€

La integración en Home Assistant de dispositivos que implementan el protocolo **Xiaomi Mijia BLE MiBeacon** se realiza mediante [**Xiaomi BLE**](https://www.home-assistant.io/integrations/xiaomi_ble/){:target="_blank"} que, a su vez, usa la integración de [**Bluetooth**](https://www.home-assistant.io/integrations/bluetooth){:target="_blank"}.

La idea es escuchar las transmisiones _Bluetooth_ que los dispositivos hacen por sí mismos, lo cual permite obtener los últimos valores del sensor sin necesidad de despertarlo del modo de suspensión profunda y conservar así su batería.

# Bluetooth

La integración [Bluetooth](https://www.home-assistant.io/integrations/bluetooth){:target="_blank"}, que se [carga por defecto](https://www.home-assistant.io/integrations/default_config/){:target="_blank"} en Home Assistant, permite detectar dispositivos cercanos que usen este protocolo.

Para disponer de _Bluetooth_ en Linux es necesario que:

* El _socket_ [D-Bus](https://en.wikipedia.org/wiki/D-Bus){:target="_blank"} debe ser accesible para Home Assistant
* El adaptador Bluetooth debe ser accesible para D-Bus y ejecutar [BlueZ](http://www.bluez.org/){:target="_blank"} >= 5.43
* La implementación de D-Bus debe ser [dbus-broker](https://github.com/bus1/dbus-broker){:target="_blank"}
* El sistema host debe ejecutar el kernel de Linux 5.15.62 o posterior

> **Nota**: En una Raspberry Pi 4 hay que comprobar que no se haya [desactivado el Bluetooth en el fichero `/boot/config.txt`](https://pimylifeup.com/raspberry-pi-disable-wifi/){:target="_blank"} mediante la directiva `dtoverlay=disable-bt`.

Para comprobar que todo está correcto, se puede ejecutar el comando `bluetoothctl`:

```
pi@pi4nas:~ $ bluetoothctl
Agent registered
```

Y también comprobar el estado del servicio `hciuart.service` mediante el comando `systemctl`:

```
pi@pi4nas:~ $ sudo systemctl status hciuart.service
● hciuart.service - Configure Bluetooth Modems connected by UART
   Loaded: loaded (/lib/systemd/system/hciuart.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2023-06-23 19:44:50 CEST; 4min 47s ago
  Process: 377 ExecStart=/usr/bin/btuart (code=exited, status=0/SUCCESS)
 Main PID: 487 (hciattach)
    Tasks: 1 (limit: 4915)
   CGroup: /system.slice/hciuart.service
           └─487 /usr/bin/hciattach /dev/serial1 bcm43xx 3000000 flow -

Jun 23 19:44:43 pi4nas systemd[1]: Starting Configure Bluetooth Modems connected by UART...
Jun 23 19:44:50 pi4nas btuart[377]: bcm43xx_init
Jun 23 19:44:50 pi4nas btuart[377]: Flash firmware /lib/firmware/brcm/BCM4345C0.hcd
Jun 23 19:44:50 pi4nas btuart[377]: Set Controller UART speed to 3000000 bit/s
Jun 23 19:44:50 pi4nas btuart[377]: Device setup complete
Jun 23 19:44:50 pi4nas systemd[1]: Started Configure Bluetooth Modems connected by UART.
```

A continuación, en Home Assistant, se puede acceder a las **integraciones** para configurarla:

* Settings
* Devices and Services
* Add Integration
* Seleccionar **Bluetooth**
* Seleccionar el dispositivo `hci0 (xx:xx:xx:xx:xx:xx)`
  * _Do you want to set up the Bluetooth adapter hci0 (xx:xx:xx:xx:xx:xx)?_
* Pulsar el botón `Submit`
* HA indicará que:
  * ha creado la configuración para `xx:xx:xx:xx:xx:xx`
  * ha encontrado el dispositivo `hci0 (xx:xx:xx:xx:xx:xx)`
* Seleccionar el área donde esté ubicado
* Pulsar el botón `Finish`

# Xiaomi BLE

La integración [Xiaomi BLE](https://www.home-assistant.io/integrations/xiaomi_ble){:target="_blank"} permite integrar dispositivos que utilizan el protocolo **Xiaomi Mijia BLE MiBeacon**.

Una vez activada, si tenemos dispositivos compatibles, puede que se hayan descubierto en:

* Settings
* Devices and Services
* Discovered
  * _Temperature/Humidity Sensor XXXX (LYWSD03MMC) Xiaomi BLE_
    * Pulsar el botón `Add`

{: .box-note}
Si aparece el siguiente mensaje informativo, se puede hacer clic en `Submit` para añadirlo:<br><br>
"_There hasn't been a broadcast from this device in the last minute so we aren't sure if this device uses encryption or not. This may be because the device uses a slow broadcast interval. Confirm to add this device anyway, then the next time a broadcast is received you will be prompted to enter its bindkey if it's needed._"

* Do you want to set up Temperature/Humidity Sensor XXXX (LYWSD03MMC)?
  * Hacer clic en `Submit`
  * Escoger el área en la que se ubica el dispositivo
  * Hacer clic en `Finish`

```
Success!
Created configuration for Temperature/Humidity Sensor XXX (LYWSD03MMC).
We found the following devices:
Temperature/Humidity Sensor XXX
LYWSD03MMC (Xiaomi)
Area: xxxx
```

{: .box-note}
Podría ser que los dispositivos se hubiesen descubierto mediante la integración [BTHome](https://www.home-assistant.io/integrations/bthome){:target="_blank"} si no se ha cambiado el tipo de anuncio de `BTHome v2` a `MIJIA (Mi Home)` al cambiar el _firmware_ de los mismos.

# Flash `atc1441` o `pvvx`

Existen dos proyectos en GitHub que permiten grabar un **_firmware_ alternativo** en estos termómetros de Xiaomi:

* [ATC MiThermometer @atc1441](https://github.com/atc1441/ATC_MiThermometer){:target="_blank"}, es el proyecto original de Aaron Christophel
* [ATC MiThermometer @pvvx](https://github.com/pvvx/ATC_MiThermometer){:target="_blank"}, es un _fork_ del anterior realizado por un ruso llamado Victor que incorpora bastantes mejoras (por ejemplo, una mejor gestión del consumo de energía)

Los dos proyectos tienen una página web que permite realizar esta grabación usando un simple navegador (Google Chrome o Microsoft Edge) desde un dispositivo con Bluetooth:

* [https://atc1441.github.io/TelinkFlasher.html](https://atc1441.github.io/TelinkFlasher.html){:target="_blank"}
* [https://pvvx.github.io/ATC_MiThermometer/TelinkMiFlasher.html](https://pvvx.github.io/ATC_MiThermometer/TelinkMiFlasher.html){:target="_blank"}

Aunque el procedimiento para _flashear_ un _firmware_ alternativo es bastante similar en los dos proyectos, es recomendable utilizar la herramienta de `pvvx`:

* Activar Bluetooth en un teléfono móvil Android
* Usar el navegador Google Chrome o Microsoft Edge
* Acceder a la página [https://pvvx.github.io/ATC_MiThermometer/TelinkMiFlasher.html](https://pvvx.github.io/ATC_MiThermometer/TelinkMiFlasher.html){:target="_blank"}
* Pulsar el botón `Connect`
* Seleccionar el dispositivo `LYWSD03MMC` y pulsar el botón `Emparejar`

{: .box-note}
Si el dispositivo ya tenía un _firmware_ alternativo, se llamará `ATC_XXXXX` en lugar de `LYWSD03MMC`.

Si el emparejamiento funciona correctamente, en la pantalla del dispositivo aparecerá el símbolo de Bluetooth para indicarlo. 

* Pulsar el botón `Custom Firmware: ATC_vXX.bin` para seleccionar la última versión del _firmware_
* Pulsar el botón `Start Flashing`

Comenzará el proceso de envío de los bloques del _firmware_ a través de BT y, si todo va bien, en unos 30 segundos estará actualizado y listo para configurar sus parámetros.

![PVVX Flash][1]

{: .box-note}
Si hubiese problemas para configurar el dispositivo en Home Assistant, se puede revisar la _issue_ "[Xiaomi BLE not asking for bindkey](https://github.com/home-assistant/core/issues/77681){:target="_blank"}" donde se explica cómo [reconfigurar el dispositivo](https://github.com/home-assistant/core/issues/77681#issuecomment-1237453198){:target="_blank"} a los 10 minutos usando _bindkey_ obtenida mediante el botón `Do Activation`.

# Configuración

Una vez se ha cambiado el _firmware_, se podrá conectar de nuevo con el dispositivo y configurarlo:

* Pulsar el botón `Get Config` para mostrar la configuración actual
* Pulsar el botón `Get Time` para mostrar la fecha y hora del dispositivo (si se ha quitado la batería suele ser 1970)
* Pulsar el botón `Set Time` para enviar la fecha y hora actual al dispositivo
* Cambiar los siguientes parámetros de la configuración:
  * Activar **Show batt** [x]
  * Activar **Sensor in "LowPower mode"** [x]
  * Cambiar el **Advertising type** de `BTHome v2` a `MIJIA (MiHome)`
  * Cambiar el **Advertising interval** de `2500.0` ms a `5000.0` ms
  * Cambiar el **Measure interval** de `4` a `12` (es decir, de 20 sec a 60 sec)
  * Cambiar el **Minimum LCD refresh rate** de `2.45` a `10.00`
* Pulsar el botón `Set Config` para enviar la configuración
* Pulsar el botón `Disconnect` para desconectarse del dispositivo

{: .box-note}
Esta configuración envia el siguiente _custom string_ en hexadecimal: `55a6100000500ca931c80ab400`

![PVVX Configuration][2]

# Zigbee

También existe un proyecto en GitHub del usuario `devbis` para convertir un `LYWSD03MMC` de BT a Zigbee:

* [Zigbee 3.0 Firmware for original LYWSD03MMC Sensor](https://github.com/devbis/z03mmc){:target="_blank"}

Este proyecto también tiene una página web [https://devbis.github.io/telink-zigbee/](https://devbis.github.io/telink-zigbee/){:target="_blank"} que permite grabar el _custom firmware_ usando un simple navegador (Google Chrome o Microsoft Edge) desde un dispositivo con Bluetooth.

Habrá que investigar esta opción para ver si el consumo de batería es menor que usando BT.

# Referencias

* [Temperatura y Humedad con Xiaomi LYWSD03MMC en Grafana](https://www.alferez.es/iot/temperatura-humedad-xiaomi-grafana-lywsd03mmc/){:target="_blank"} @ Blog Tecnológico de Alférez
* [Convert Xiaomi LYWSD03MMC From Bluetooth to Zigbee](https://smarthomescene.com/guides/convert-xiaomi-lywsd03mmc-from-bluetooth-to-zigbee/){:target="_blank"} @ SmartHomeScene
* [4$ Xiaomi thermometer custom firmware LYWSD03MMC BLE TLSR8251](https://youtu.be/NXKzFG61lNs){:target="_blank"} @ YouTube (Aaron Christophel, atc1441)
* [Vídeo: Tutorial 19 – Cambio a Custom Firmware del Termohigrómetro Xiaomi LYWSD03MMC](https://domoticaencasa.es/tutorial-custom-firmware-termohigrometro-xiaomi-lywsd03mmc/){:target="_blank"} @ Domótica en Casa
* [Xiaomi Temperature & Humidity Sensor Home Assistant Integration](https://www.simplysmart.house/blog/xiaomi-temprature-home-asssistant-integration){:target="_blank"} @ Simply Smart House
* [Home Asssistant Xiaomi Mijia LYWSD03MMC Temperature and Humidity Sensor Tutorial September 2022](https://community.home-assistant.io/t/home-asssistant-xiaomi-mijia-lywsd03mmc-temperature-and-humidity-sensor-tutorial-september-2022/456403){:target="_blank"}


### Historial de cambios

* **2023-06-22**: Notas iniciales
* **2025-01-12**: Revisión y publicación del documento

[1]: /assets/img/blog/2023-06-22_image_1.png "PVVX Flash"
[2]: /assets/img/blog/2023-06-22_image_2.png "PVVX Configuration"