---
layout : post
blog-width: true
title: 'Instalación de Zigbee2MQTT en Docker'
date: '2023-03-05 13:21:27'
published: true
tags:
- Home Assistant
- Docker
author:
  display_name: Manel Rodero
---

# Introducción a Zigbee

Se llama [**Zigbee**](https://es.wikipedia.org/wiki/Zigbee) a un conjunto de protocolos de comunicación inalámbrica muy utilizado en dispositivo domóticos debido a su bajo consumo, su topología de red en malla y su bajo coste de integración.

Empezar a montar una red Zigbee puede ser una autentica pesadilla debido a la existencia de múltiples tipos de dispositivos en función del papel que juegan en la red:

* Coordinador Zigbee (_Zigbee Coordinator_, ZC)
* Router Zigbee (_Zigbee Router_, ZR)
* Dispositivo final (_Zigbee End Device_, ZED)

Por ejemplo, a la hora de elegir un _hub_ Zigbee encontramos diferentes marcas como Tuya, Xiaomi, Aqara, Sonoff, Philips (Hue), Amazon (Alexa), etc.:

![Zigbee Hub][1]

Aunque puede ser cómodo utilizar uno de estos aparatos, en la práctica limitan lo que se puede llegar a hacer porque suelen funcionar únicamente con dispositivos de su misma marca o necesitan acceder a un _cloud_ propietario para funcionar.

Una alternativa es utilizar algún desarrollo _open source_ que permita montar una red Zigbee integrada en [Home Assistant](instalacion-de-home-assistant-en-docker), por ejemplo:

* [Zigbee2MQTT](https://www.zigbee2mqtt.io/)
* [Zigbee Home Automation (ZHA)](https://www.home-assistant.io/integrations/zha/)

Las ventajas de Zigbee2MQTT sobre el módulo oficial ZHA de Home Assistant son:

* Es un proyecto con mucha comunidad detrás y muy estable
* Soporta por defecto muchos dispositivos Zigbee baratos
* Se puede añadir fácilmente un dispositivo no soportado
* Los _stick_ **CC2652** son muy baratos y funcionan muy bien
* Al no estar acoplado con HA:
  * la red Zigbee no se reinicializa cuando lo hace HA
  * permite añadir a la red dispositivos que no estén en HA
* Se [integra fácilmente con HA](https://www.zigbee2mqtt.io/guide/usage/integrations/home_assistant.html) utilizando [Zigbee2Mqtt Assistant](https://github.com/yllibed/Zigbee2MqttAssistant)

# Zigbee2MQTT

[Zigbee2MQTT](https://www.zigbee2mqtt.io/) es un _bridge_ Zigbee que no necesita acceso a un _cloud_ propietario para funcionar.

Para poder utilizarlo se necesita el siguiente _hardware_:

* Un [adaptador Zigbee](https://www.zigbee2mqtt.io/guide/adapters/) para comunicarse con la red Zigbee
  * En mi caso usaré un [SONOFF ZigBee 3.0 USB Dongle Plus](https://sonoff.tech/product/gateway-amd-sensors/sonoff-zigbee-3-0-usb-dongle-plus-p/) que se puede comprar fácilmente en [AliExpress](https://es.aliexpress.com/item/1005003758328408.html) por unos 23€
* Un servidor donde ejecutarlo
  * En mi caso usaré una Raspberry Pi 4 en la que se ha instalado el _broker_ MQTT [Mosquitto](instalacion-de-mosquitto-mqtt-broker-en-docker)
* Uno o más [dispositivos Zigbee](https://www.zigbee2mqtt.io/supported-devices/) para añadirlos a la red

> **Nota**: A día de hoy, Zigbee2MQTT soporta 2786 dispositivos de 359 fabricantes diferentes.

```
Zigbee 3.0 USB Dongle Plus
Model: ZBDongle-P
Input: 5V, 100mA Max
Working temperature: -10°C a 40°C
Wireless connection: Zigbee 3.0
Dimensions: 87mm x 25,5mm x 13,5mm
```

# Instalación

La instalación de [Zigbee2MQTT en Docker](https://www.zigbee2mqtt.io/guide/installation/02_docker.html) se realiza usando la imagen [`koenkk/zigbee2mqtt`](https://hub.docker.com/r/koenkk/zigbee2mqtt/).

La forma más sencilla de hacerlo es usando un fichero `docker-compose.yml` con el siguiente contenido:

```yaml
version: '3.8'
services:
  zigbee2mqtt:
    image: koenkk/zigbee2mqtt
    container_name: zigbee2mqtt
    environment:
      - TZ=Europe/Madrid
    volumes:
      - /home/pi/volumes/zigbee2mqtt:/app/data
      - /run/udev:/run/udev:ro
    ports:
      - 1881:8080
    restart: unless-stopped
    devices:
      # Make sure this matched your adapter location
      # - /dev/ttyUSB0:/dev/ttyACM0
      - /dev/serial/by-id/usb-Silicon_Labs_Sonoff_Zigbee_3.0_USB_Dongle_Plus_0001-if00-port0:/dev/ttyACM0
```

> **Nota**: para identificar el dispositivo que se ha conectado se puede utilizar el comando `ls -l /dev/serial/by-id` tal como se muestra a continuación:

```
pi@raspberry:~ $ ls -l /dev/serial/by-id
total 0
lrwxrwxrwx 1 root root 13 Dec  1 19:28 usb-Silicon_Labs_Sonoff_Zigbee_3.0_USB_Dongle_Plus_0001-if00-port0 -> ../../ttyUSB0
```

Para iniciar el contenedor, se puede usar `docker-compose up -d` o usar el contenido del fichero anterior en **Portainer**.

# Configuración

Una vez iniciado el contenedor, se pueden revisar su _logs_ utilizando el comando `docker logs -f zigbee2mqtt`.

Si aparece el error `network commissioning timed out` al iniciarlo, lo más probable es que se deba a las interferencias de radio que se producen al conectar el _dongle_ directamente a un puerto USB de la Raspberry Pi.

La "solución" es utilizar un [cable extensor USB](https://github.com/Koenkk/zigbee2mqtt/issues/10209) para conectar el _dongle_ y alejarlo. Además, esta opción permite colocarlo a más altura para aumentar la distancia de transmisión.

## Autenticación

En este fragmento de _log_ se puede observar la **versión del coordinador** `20210708` (que no es la más reciente a día de hoy) y algunos errores de conexión a MQTT debidos a la [autenticación de Mosquitto](instalacion-de-mosquitto-mqtt-broker-en-docker):


```
Zigbee2MQTT:info  2023-03-05 16:37:08: Starting Zigbee2MQTT version 1.30.2 (commit #cdf62ea)
Zigbee2MQTT:info  2023-03-05 16:37:08: Starting zigbee-herdsman (0.14.96)
Zigbee2MQTT:info  2023-03-05 16:37:09: zigbee-herdsman started (resumed)
Zigbee2MQTT:info  2023-03-05 16:37:09: Coordinator firmware version: '{"meta":{"maintrel":1,"majorrel":2,"minorrel":7,"product":1,"revision":20210708,"transportrev":2},"type":"zStack3x0"}'
Zigbee2MQTT:info  2023-03-05 16:37:09: Currently 0 devices are joined:
Zigbee2MQTT:warn  2023-03-05 16:37:09: `permit_join` set to  `true` in configuration.yaml.
Zigbee2MQTT:warn  2023-03-05 16:37:09: Allowing new devices to join.
Zigbee2MQTT:warn  2023-03-05 16:37:09: Set `permit_join` to `false` once you joined all devices.
Zigbee2MQTT:info  2023-03-05 16:37:09: Zigbee: allowing new devices to join.
Zigbee2MQTT:info  2023-03-05 16:37:09: Connecting to MQTT server at mqtt://localhost
Zigbee2MQTT:error 2023-03-05 16:37:10: MQTT error: connect ECONNREFUSED 127.0.0.1:1883
Zigbee2MQTT:error 2023-03-05 16:37:10: MQTT failed to connect, exiting...
Zigbee2MQTT:info  2023-03-05 16:37:10: Stopping zigbee-herdsman...
```

Se puede modificar el fichero `/home/pi/volumes/zigbee2mqtt/configuration.yaml` y añadir el **usuario** y **contraseña** para conectarse a MQTT:

```yaml
mqtt:
  base_topic: zigbee2mqtt
  server: 'mqtt://192.168.1.180:1883'
  user: admin
  password: CHdDECZrt0GONub3JtlkC0IGq
```

Si todo es correcto, los _logs_ mostrarán el inicio correcto de **Zigbee2MQTT**:

```
Zigbee2MQTT:info  2023-03-05 17:03:20: Starting Zigbee2MQTT version 1.30.2 (commit #cdf62ea)
Zigbee2MQTT:info  2023-03-05 17:03:20: Starting zigbee-herdsman (0.14.96)
Zigbee2MQTT:info  2023-03-05 17:03:21: zigbee-herdsman started (resumed)
Zigbee2MQTT:info  2023-03-05 17:03:21: Coordinator firmware version: '{"meta":{"maintrel":1,"majorrel":2,"minorrel":7,"product":1,"revision":20210708,"transportrev":2},"type":"zStack3x0"}'
Zigbee2MQTT:info  2023-03-05 17:03:21: Currently 0 devices are joined:
Zigbee2MQTT:warn  2023-03-05 17:03:21: `permit_join` set to  `true` in configuration.yaml.
Zigbee2MQTT:warn  2023-03-05 17:03:21: Allowing new devices to join.
Zigbee2MQTT:warn  2023-03-05 17:03:21: Set `permit_join` to `false` once you joined all devices.
Zigbee2MQTT:info  2023-03-05 17:03:21: Zigbee: allowing new devices to join.
Zigbee2MQTT:info  2023-03-05 17:03:22: Connecting to MQTT server at mqtt://192.168.1.180:1883
Zigbee2MQTT:info  2023-03-05 17:03:22: Connected to MQTT server
Zigbee2MQTT:info  2023-03-05 17:03:22: MQTT publish: topic 'zigbee2mqtt/bridge/state', payload '{"state":"online"}'
Zigbee2MQTT:info  2023-03-05 17:03:22: Zigbee2MQTT started!
```

## Clave de red

A continuación hay que [cambiar la **clave de red**](https://www.zigbee2mqtt.io/guide/configuration/zigbee-network.html#network-config) para evitar el uso de la clave por defecto. Esto se consigue añadiendo `network_key: GENERATE` en la sección `advanced` y reiniciando el contenedor.

## Frontend

Se puede habilitar un _frontend_ muy sencillo añadiendo `frontend: true` al fichero de configuración y reiniciando el contenedor.

Una vez reiniciado, se puede acceder al panel de configuración a través del puerto `1881` (en este ejemplo [http://192.168.1.180:1881](http://192.168.1.180:1881)).

![Frontend Zigbee2MQTT][2]

## Integración con Home Assistant

La forma más sencilla para [integrar Zigbee2MQTT en Home Assistant](https://www.zigbee2mqtt.io/guide/configuration/homeassistant.html) mediante **MQTT Discovery** es añadir `homeassistant: true` al fichero de configuración y reiniciar el contenedor.

> **Nota**: Para que **MQTT Discovery** funcione correctamente hay que [habilitar la integración MQTT en Home Assistant](https://www.home-assistant.io/integrations/mqtt/).

Se recomienda leer la [guía de integración](https://www.zigbee2mqtt.io/guide/usage/integrations/home_assistant.html) para personalizar el descubrimiento de dispositivos y su configuración automática en Home Assistant.

# Agregar dispositivos

En este ejemplo se añadirá un sensor de temperatura y humedad [**SONOFF SNZB-02**](https://sonoff.tech/product/gateway-and-sensors/snzb-02/) que se puede comprar fácilmente en [AliExpress](https://es.aliexpress.com/item/4001278991530.html) por unos 10€.

```
Temperature and Humidity Sensor
Model: SNZB-02
Battery model: CR2450 (3V)
Working temperature: -10°C a 40°C
Working humidity: 10-90%RH (non-condensing)
Wireless connection: Zigbee (IEEE 802.15.4)
Dimensions: 43mm x 43mm x 14mm
```

El sensor [SNZB-02 está soportado](https://www.zigbee2mqtt.io/devices/SNZB-02.html) y la forma de realizar el **emparejamiento** (_pairing_) con la red ZigBee es la siguiente:

* Comprobar que Zigbee2MQTT permite la unión de dispositivos
* Abrir la tapadera del sensor (puede costar un poco)
* Retirar el plástico aislante de la batería
* Volver a colocar la tapadera
* Pulsar el botón durante 5 segundos hasta que el indicator LED parpadee 3 veces (significa que ha entrado en modo _pairing_)

Si todo funciona correctamente, en los _logs_ aparecerá el nuevo dispositivo:

```
Zigbee2MQTT:info  2023-03-05 18:56:37: Zigbee: allowing new devices to join.
Zigbee2MQTT:info  2023-03-05 18:56:37: MQTT publish: topic 'zigbee2mqtt/bridge/response/permit_join', payload '{"data":{"time":254,"value":true},"status":"ok","transaction":"etfxo-2"}'
Zigbee2MQTT:info  2023-03-05 18:57:01: Device '0x00124b0012345678' joined
Zigbee2MQTT:info  2023-03-05 18:57:01: MQTT publish: topic 'zigbee2mqtt/bridge/event', payload '{"data":{"friendly_name":"0x00124b0012345678","ieee_address":"0x00124b0012345678"},"type":"device_joined"}'
Zigbee2MQTT:info  2023-03-05 18:57:01: Starting interview of '0x00124b0012345678'

[...]

Zigbee2MQTT:info  2023-03-05 18:57:22: Successfully interviewed '0x00124b0012345678', device has successfully been paired
Zigbee2MQTT:info  2023-03-05 18:57:22: Device '0x00124b0012345678' is supported, identified as: SONOFF Temperature and humidity sensor (SNZB-02)
Zigbee2MQTT:info  2023-03-05 18:57:22: MQTT publish: topic 'zigbee2mqtt/bridge/event', payload '{"data":{"definition":{"description":"Temperature and humidity sensor","exposes":[{"access":1,"description":"Remaining battery in %, can take up to 24 hours before reported.","name":"battery","property":"battery","type":"numeric","unit":"%","value_max":100,"value_min":0},{"access":1,"description":"Measured temperature value","name":"temperature","property":"temperature","type":"numeric","unit":"°C"},{"access":1,"description":"Measured relative humidity","name":"humidity","property":"humidity","type":"numeric","unit":"%"},{"access":1,"description":"Voltage of the battery in millivolts","name":"voltage","property":"voltage","type":"numeric","unit":"mV"},{"access":1,"description":"Link quality (signal strength)","name":"linkquality","property":"linkquality","type":"numeric","unit":"lqi","value_max":255,"value_min":0}],"model":"SNZB-02","options":[{"access":2,"description":"Number of digits after decimal point for temperature, takes into effect on next report of device.","name":"temperature_precision","property":"temperature_precision","type":"numeric","value_max":3,"value_min":0},{"access":2,"description":"Calibrates the temperature value (absolute offset), takes into effect on next report of device.","name":"temperature_calibration","property":"temperature_calibration","type":"numeric"},{"access":2,"description":"Number of digits after decimal point for humidity, takes into effect on next report of device.","name":"humidity_precision","property":"humidity_precision","type":"numeric","value_max":3,"value_min":0},{"access":2,"description":"Calibrates the humidity value (absolute offset), takes into effect on next report of device.","name":"humidity_calibration","property":"humidity_calibration","type":"numeric"}],"supports_ota":false,"vendor":"SONOFF"},"friendly_name":"0x00124b0012345678","ieee_address":"0x00124b0012345678","status":"successful","supported":true},"type":"device_interview"}'
Zigbee2MQTT:info  2023-03-05 18:57:22: Configuring '0x00124b0012345678'

[...]

Zigbee2MQTT:info  2023-03-05 18:57:26: MQTT publish: topic 'zigbee2mqtt/0x00124b0012345678', payload '{"battery":null,"humidity":null,"linkquality":141,"temperature":null,"voltage":3400}'
```

El dispositivo también aparecerá identificado en el panel de configuración:

![SONOFF SNZB-02][3]

y se podrá acceder a sus características (fabricante, modelo, dirección IEEE, etc.), valores que **expone** (batería, temperatura, humedad, etc.) y el intervalo de notificación de los mismos:

![Valores expuestos][4]

Finalmente, si la integración con Home Assistant está bien realizada, también aparecerá el dispositivo en MQTT:

![Home Assistant][5]

# Actualización firmware

Mantener actualizado el _firmware_ del **coordinador Zigbee** es muy importante para que **Zigbee2MQTT** funcione correctamente, sobretodo con dispositivos modernos.

## Descarga del software

* Desde la la página de [adaptadores recomendados](https://www.zigbee2mqtt.io/guide/adapters/#recommended) de Zigbee2MQTT, seleccionar **SONOFF Zigbee 3.0 USB Dongle Plus ZBDongle-P** para desplegar los enlaces de este dispositivo
* Hacer clic en el enlace para descargar el _firwmare_ del coordinador [`CC1352P2_CC2652P_launchpad_coordinator_20221226.zip`](https://github.com/Koenkk/Z-Stack-firmware/raw/master/coordinator/Z-Stack_3.x.0/bin/CC1352P2_CC2652P_launchpad_coordinator_20221226.zip)
* En la sección [`Flashing CC1352/CC2652/CC2538 based adapters`](https://www.zigbee2mqtt.io/guide/adapters/#notes) seguimos el enlace a la página de la herramienta [**ZigStar GW Multi tool**](https://github.com/xyzroe/ZigStarGW-MT)
* Descargar la última _release_ [`ZigStarGW-MT-x64.exe.zip`](https://github.com/xyzroe/ZigStarGW-MT/releases/download/v0.3.5/ZigStarGW-MT-x64.exe.zip) para Windows y descomprimirla
* Desde la página [CP210x USB to UART Bridge Virtual COM Port (VCP)](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers) descargar la versión `11.2.0` o superior del [CP210x Universal Windows Driver](https://www.silabs.com/documents/public/software/CP210x_Universal_Windows_Driver.zip)

## Preparación del _dongle_

Para actualizar el _firmware_ es necesario abrir el _dongle_ ya que se necesita pulsar el botón `BOOT` para ponerlo en el modo correcto.

* Quitar la antena del _dongle_
* Quitar los dos pequeños tornillos y la tapa del lado de la antena
* Sacar la placa y observar los dos botones `BOOT` y `RST`
* Conectar la placa al puerto USB de un ordenador Windows **al mismo que se pulsa el botón `BOOT`**

> **Nota**: si el _dongle_ aparece en la sección `Other devices` del Administrador de Dispositivos significa que no están instalados los _drivers_ UART que hemos descargado anteriormente.

![Other devices][6]

![UART Bridge][7]

## _Flashing_

El proceso de actualización del _firmware_ es bastante sencillo pero puede ser necesario un pequeño "truco" si aparece algún error:

* Ejecutar la herramienta `ZigStarGW-MT`
* Seleccionar el puerto COM donde está el dispositivo (en este ejemplo es `COM3`)
* Cargar el fichero `` con el _firmware_
* Marcar las opciones `Erase`, `Write` y `Verify`
* Pulsar el botón `Start`
* No desconectar la placa mientras se actualiza ;-)

> **Nota**: si aparece un error de _timeout_ al esperar el ACK, se desconectará el _dongle_ y se volverá a conectar de la misma manera, manteniendo pulsado el botón `BOOT` hasta que empiece el proceso de actualización después de pulsar el botón `Start`.

![ZigStar GW Multi Tool][8]

## Montaje del _dongle_

Finalmente se procede al montaje del _dongle_ para ser utilizado de nuevo:

* Desconectar la placa del puerto USB
* Introducir la placa en su caja metálica
* Volver a colocar la tapa y los dos pequeños tornillos
* Volver a colocar la antena
* Conectar de nuevo el _dongle_ en la Raspberry Pi

Een los _logs_ de Zigbee2MQTT se podrá observar la versión del _firmware_ instalada, en este caso la **2022126**:

```
Zigbee2MQTT:info  2023-03-05 21:00:54: Starting Zigbee2MQTT version 1.30.2 (commit #cdf62ea)
Zigbee2MQTT:info  2023-03-05 21:00:54: Starting zigbee-herdsman (0.14.96)
Zigbee2MQTT:info  2023-03-05 21:01:33: zigbee-herdsman started (restored)
Zigbee2MQTT:info  2023-03-05 21:01:33: Coordinator firmware version: '{"meta":{"maintrel":1,"majorrel":2,"minorrel":7,"product":1,"revision":20221226,"transportrev":2},"type":"zStack3x0"}'
```

# Actualizar

Si ya se había instalado Mosquitto, se puede actualizar de la siguiente manera:

```
docker stop zigbee2mqtt
docker rm zigbee2mqtt
docker rmi zigbee2mqtt
docker-compose up -d
```

# Soporte

Algunos comandos para gestionar la configuración del contenedor:

```
# Acceder al shell mientras el contenedor está ejecutándose
docker exec -it zigbee2mqtt /bin/sh

# Monitorizar los logs del contenedor en tiempo real
docker logs -f zigbee2mqtt
```

# Referencias

* [Zigbee 3.0 USB Dongle Plus–ZBDongle-E](https://itead.cc/product/zigbee-3-0-usb-dongle/)

* Playlist [Zigbee2MQTT / Zigbee](https://www.youtube.com/playlist?list=PLMuAf-x9dLNgQMQMk_H1X_RSSbS_n5bvK) @ Un loco y su tecnología
  * [Empezar con Zigbee y Home Assistant: ¿Qué opciones hay? Zigbee2MQTT / ZHA / deCONZ - Capitulo 1](https://www.youtube.com/watch?v=FGTLpdgvfPI)
  * [Instala Zigbee2MQTT y conecta todos tus sensores a Home Assistant - Capitulo 2](https://www.youtube.com/watch?v=6ZhRWMtbXqo)
  * [Cómo añadir dispositivos a Zigbee2MQTT y Home Assistant? Capítulo 3](https://www.youtube.com/watch?v=kQCE5h4LwYE)
  * [Zigbee2MQTT bajo Home Assistant OS - Instalación como Add-On ](https://www.youtube.com/watch?v=VUwu2QQmXfU)
  * [Zigbee2MQTT bajo Home Assistant OS (Mayo 2022)](https://www.youtube.com/watch?v=UtobiuUxvqc)
  * [Actualiza el firmware del Dongle SONOFF Zigbee 3.0 SIN MIEDO](https://www.youtube.com/watch?v=jycS_HSgz6A)

* Playlist [eWeLink Cast](https://www.youtube.com/playlist?list=PL3cdVloppBaxcGn7EjAMPVV-_HJq6DyqZ) @ eWeLink Smart Home

[1]: /assets/img/blog/2023-03-05_image_1.png "Zigbee Hubs"
[2]: /assets/img/blog/2023-03-05_image_2.png "Frontend Zigbee2MQTT"
[3]: /assets/img/blog/2023-03-05_image_3.png "SONOFF SNZB-02"
[4]: /assets/img/blog/2023-03-05_image_4.png "Valores expuestos"
[5]: /assets/img/blog/2023-03-05_image_5.png "Home Assistant"
[6]: /assets/img/blog/2023-03-05_image_6.png "Other devices"
[7]: /assets/img/blog/2023-03-05_image_7.png "UART Bridge"
[8]: /assets/img/blog/2023-03-05_image_8.png "ZigStar GW Multi Tool"