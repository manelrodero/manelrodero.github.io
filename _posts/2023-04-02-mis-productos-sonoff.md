---
layout : post
blog-width: true
title: 'Mis productos SONOFF'
date: '2023-04-02 17:59:15'
published: true
tags:
- Home Assistant
- Docker
author:
  display_name: Manel Rodero
---

# ¿Qué es SONOFF?

[SONOFF](https://sonoff.tech/) es una gama de productos para el "hogar inteligente" de la empresa [ITEAD Intelligent Systems Co., Ltd](https://itead.cc/), una plataforma internacional de comercio de productos electrónicos que también ofrece pantallas HMI [Nextion](https://nextion.tech/), receptores SDR [Airspy](https://airspy.com/), etc.

# Gateway & Sensors

## [Zigbee 3.0 USB Dongle Plus-P](https://sonoff.tech/product/gateway-amd-sensors/sonoff-zigbee-3-0-usb-dongle-plus-p/)

El **ZigBee 3.0 USB Dongle Plus** es un adaptador USB que se puede utilizar como una puerta de enlace Zigbee universal en Home Assistant] a través de [Zigbee2MQTT](https://www.zigbee2mqtt.io/devices/ZBDongle-E.html) para **controlar localmente todos los dispositivos Zigbee** sin necesidad de comprar concentradores Zigbee de diferentes marcas.

Se puede comprar en [AliExpress](https://es.aliexpress.com/item/1005003606767695.html) por unos 22,50€ (o 25,32€ con un cable de extensión USB de 1,5m) a fecha 14 de abril de 2023.

![ZBDongle-P](https://www.zigbee2mqtt.io/images/devices/ZBDongle-E.jpg)

Sus [especificaciones](https://sonoff.tech/wp-content/uploads/2021/11/%E4%BA%A7%E5%93%81%E5%8F%82%E6%95%B0%E8%A1%A8-ZBDongle-P-20211008.pdf) son las siguientes:

| Model | ZBDongle-P |
| Product series | DIY smart switch |
| Type | DIY universal gateway |
| Input | 5VDC 100mA Max |
| MCU | CC2652P |
| Wireless connection | Zigbee 3.0 (IEEE 802.15.4) |
| Antenna | External antenna |
| Security mode | WPA/WPA2 |
| Encryption type | AES-128 |
| Applicable place | Indoor |
| Working temperature | -10°C ~ 40°C |
| Working humidity | 5-95%RH (non-condensing) |
| Working height | Less than 2000m |
| Dimensions | 87mm x 25,5mm x 13,5mm |

## [SNZB-02](https://sonoff.tech/product/gateway-and-sensors/snzb-02/) ZigBee Temperature & Humidity Sensor

El **SNZB-02** es un sensor Zigbee que puede emplearse para monitorizar la temperatura y humedad del entorno en tiempo real. Al ser Zigbee tiene un bajo consumo de energía y se puede integrar fácilmente en Home Assistant usando [Zigbee2MQTT](https://www.zigbee2mqtt.io/devices/SNZB-02.html).

Se puede comprar en [AliExpress](https://es.aliexpress.com/item/4001279272382.html) por 9,60€ (incluyendo la batería) a fecha 14 de abril de 2023.

![SZNB-02](https://www.zigbee2mqtt.io/images/devices/SNZB-02.jpg)

Sus [especificaciones](https://sonoff.tech/wp-content/uploads/2021/03/%E4%BA%A7%E5%93%81%E5%8F%82%E6%95%B0%E8%A1%A8-SNZB-02-20201116.pdf) son las siguientes:

| Model | SNZB-02 |
| Product series | Smart Home Security |
| Type | Sensor |
| Detection ramges | Temperature and humidity |
| Battery model | CR2450 (3V) |
| Battery lifespan | >1 year |
| MCU | CC2530 |
| Wireless connection | Zigbee (IEEE 802.15.4) |
| Security mode | WPA/WPA2 |
| Encryption type | AES-128 |
| Applicable place | Indoor |
| Working temperature | -10°C ~ 40°C |
| Working humidity | 10-95%RH (non-condensing) |
| Working height | Less than 2000m |
| Dimensions | 43mm x 43mm x 14mm |

### Primer uso

Para usar el dispositivo por primera vez hay que seguir las instrucciones del [manual de usuario](https://sonoff.tech/product-document/gateway-and-sensors-doc/snzb-02-doc/):

* Abrir la caja usando un pequeño destornillador en las hendiduras laterales
* Quitar el papel que evita que la batería haga contacto
* Colocar bien la batería y ajustarla para evitar que se mueva
* En Zigbee2MQTT pulsar el botón `Permit join (All)`
* Pulsar el botón _reset_ del sensor durante **5 segundos** usando un pequeño alfiler
* El indicador LED parpadeará 3 veces y el sensor entrará en modo _pairing_
* Si todo va bien, en Zigbee2MQTT debería aparecer un dispositivo nuevo y producirse los siguientes eventos:
  * unión del dispositivo
  * proceso de _interview_ del mismo
  * emparejamiento (`zigbee2mqtt/bridge/event`)
  * configuración
  * agregación a Home Assistant (`homeassistant/binary_sensor`).
* En ZigbeeMQTT se accede a la pestaña `Devices` y se selecciona el dispositivo
  * Se accede a la pestaña `Exposes` donde se visualizan los valores `temperature` y `humidity`

![SNZB-02][1]

## [SNZB-03](https://sonoff.tech/product/gateway-and-sensors/snzb-03/) Zigbee Motion Sensor

El **SNZB-03** es un sensor Zigbee que puede detectar el movimiento de objetos en tiempo real. Al ser Zigbee tiene un bajo consumo de energía y se puede integrar fácilmente en Home Assistant usando [Zigbee2MQTT](https://www.zigbee2mqtt.io/devices/SNZB-03.html).

Se puede comprar en [AliExpress](https://es.aliexpress.com/item/1005001275307214.html) por 10,75€ (incluyendo la batería) a fecha 14 de abril de 2023.

![SZNB-03](https://www.zigbee2mqtt.io/images/devices/SNZB-03.jpg)

Sus [especificaciones](https://sonoff.tech/wp-content/uploads/2021/03/%E4%BA%A7%E5%93%81%E5%8F%82%E6%95%B0%E8%A1%A8-SNZB-03-20201116.pdf) son las siguientes:

| Model | SNZB-03 |
| Product series | Smart Home Security |
| Type | Sensor |
| Detection distance | 6m |
| Detection angle | 110° |
| Battery model | CR2450 (3V) |
| Battery lifespan | >1 year |
| MCU | CC2530 |
| Wireless connection | Zigbee (IEEE 802.15.4) |
| Security mode | WPA/WPA2 |
| Encryption type | AES-128 |
| Applicable place | Indoor |
| Working temperature | -10°C ~ 40°C |
| Working humidity | 10-95%RH (non-condensing) |
| Working height | Less than 2000m |
| Dimensions | 40mm x 35mm x 28mm |

### Primer uso

Para usar el dispositivo por primera vez hay que seguir las instrucciones del [manual de usuario](https://sonoff.tech/product-document/gateway-and-sensors-doc/snzb-03-doc/):

* Abrir la caja usando un pequeño destornillador en las hendiduras laterales
* Quitar el papel que evita que la batería haga contacto
* Colocar bien la batería y ajustarla para evitar que se mueva
* Cerrar la caja poniendo atención a la dirección de las flechas
* En Zigbee2MQTT pulsar el botón `Permit join (All)`
* Pulsar el botón _reset_ del sensor durante **5 segundos** usando un alfiler en el pequeño orificio
* El indicador LED parpadeará 3 veces y el sensor entrará en modo _pairing_
* Si todo va bien, en Zigbee2MQTT debería aparecer un dispositivo nuevo y producirse los siguientes eventos:
  * unión del dispositivo
  * proceso de _interview_ del mismo
  * emparejamiento (`zigbee2mqtt/bridge/event`)
  * configuración
  * agregación a Home Assistant (`homeassistant/binary_sensor`).
* En ZigbeeMQTT se accede a la pestaña `Devices` y se selecciona el dispositivo
  * Se accede a la pestaña `Exposes` donde se visualiza el valor `occupancy`
  * Ubicar el dispositivo en su posición y no moverlo
  * Si no hay movimiento el estado debería cambiar a `Clear`
  * Si hay movimiento el estado debería cambiar a `Occupied`

![SNZB-03][2]

## [SNZB-04](https://sonoff.tech/product/gateway-and-sensors/snzb-04/) ZigBee Wireless Door/Window Sensor

El **SNZB-04** es un sensor Zigbee que permite detectar la apertura/cierre de puertas y ventanas al separarse un imán del transmisor. Al ser Zigbee tiene un bajo consumo de energía y se puede integrar fácillmente en Home Assistant usando [Zigbee2MQTT](https://www.zigbee2mqtt.io/devices/SNZB-04.html).

Se puede comprar en [AliExpress](https://es.aliexpress.com/item/1005001275294956.html) por 9,32€ (incluyendo la batería) a fecha 14 de abril de 2023.

![SZNB-04](https://www.zigbee2mqtt.io/images/devices/SNZB-04.jpg)

Sus [especificaciones](https://sonoff.tech/wp-content/uploads/2021/03/%E4%BA%A7%E5%93%81%E5%8F%82%E6%95%B0%E8%A1%A8-SNZB-03-20201116.pdf) son las siguientes:

| Model | SNZB-04 |
| Product series | Smart Home Security |
| Type | Door/Window Sensor |
| Battery model | CR2032 (3V) |
| MCU | CC2530 |
| Wireless connection | Zigbee (IEEE 802.15.4) |
| Security mode | WPA/WPA2 |
| Encryption type | AES-128 |
| Applicable place | Indoor |
| Working temperature | -10°C ~ 40°C |
| Working humidity | 5-95%RH (non-condensing) |
| Working height | Less than 2000m |
| Installation gap | <10mm |
| Dimensions (Transmitter) | 47mm x 27mm x 13,5mm |
| Dimesions (Magnet) | 32mm x 15,6mm x 13mm |

### Primer uso

Para usar el dispositivo por primera vez hay que seguir las instrucciones del [manual de usuario](https://sonoff.tech/product-document/gateway-and-sensors-doc/snzb-04-doc/):

* Abrir la caja usando un pequeño destornillador en la hendidura del lado contrario al que tiene un pequeño orificio (botón _reset_)
* Quitar el papel que evita que la batería haga contacto
* Colocar bien la batería y ajustar la presión del soporte metálico para evitar que se mueva (se puede colocar un papel si es necesario)
* En Zigbee2MQTT pulsar el botón `Permit join (All)`
* Pulsar el botón _reset_ del sensor durante **5 segundos** usando un pequeño alfiler
* El indicador LED parpadeará 3 veces y el sensor entrará en modo _pairing_
* Si todo va bien, en Zigbee2MQTT debería aparecer un dispositivo nuevo y producirse los siguientes eventos:
  * unión del dispositivo
  * proceso de _interview_ del mismo
  * emparejamiento (`zigbee2mqtt/bridge/event`)
  * configuración
  * agregación a Home Assistant (`homeassistant/binary_sensor`).
* En ZigbeeMQTT se accede a la pestaña `Devices` y se selecciona el dispositivo
  * Se accede a la pestaña `Exposes` donde se visualiza el valor `contact`
  * Si se acerca el imán a menos de 10mm del transmisor, el estado debería cambiar a `Closed`
  * Si se aleja el imán a más de 10mm del transmisor, el estado debería cambiar a `Open`

![SNZB-04][3]

# Referencias

* [Instalación de Home Assistant en Docker](instalacion-de-home-assistant-en-docker)
* [Instalación de Zigbee2MQTT en Docker](instalacion-de-zigbee2mqtt-en-docker)

[1]: /assets/img/blog/2023-04-02_image_1.png "SNZB-02"
[2]: /assets/img/blog/2023-04-02_image_2.png "SNZB-03"
[3]: /assets/img/blog/2023-04-02_image_3.png "SNZB-04"
