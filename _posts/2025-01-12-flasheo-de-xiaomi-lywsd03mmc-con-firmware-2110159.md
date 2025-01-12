---
layout : post
blog-width: true
title: 'Flasheo de Xiaomi LYWSD03MMC con firmware 2.1.1.0159'
date: '2025-01-12 13:55:27'
#last-updated: '2025-01-12 13:55:27'
published: true
tags:
- SmartHome
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2025-01-12_cover.png"
thumbnail-img: ""
---

Hace tiempo expliqué [cómo cambiar el _firmware_ de un Xiaomi LYWSD03MMC](temperatura-y-humedad-en-home-assistant-con-xiaomi-lywsd03mmc), un monitor de temperatura y humedad, para poder integrarlo en Home Assistant.

La idea era usar la herramienta [Telink Mi Flasher](https://pvvx.github.io/ATC_MiThermometer/TelinkMiFlasher.html){:target="_blank"} para _flashear_ un _custom firmware_ utilizando un simple navegador y una conexión BT con el dispositivo.

Desde Octubre de 2023, este dispositivo tiene un _firmware_ **2.1.1_0159** mediante el cual [Xiaomi ha bloqueado la posibilidad de cambiar el _firmware_ usando Telink Mi Flasher](https://github.com/atc1441/ATC_MiThermometer/issues/298){:target="_blank"} y `pvvx` ha tenido que poner un mensaje en su herramienta:

![Firmware 2.1.1_0159][1]

# _Flash_ mediante USB-COM

A día de hoy, Enero de 2025, la única opción para grabar un _custom firmware_ es usar la herramienta [**TLSR825x USB-COM Flash Writer v0.5**](https://pvvx.github.io/ATC_MiThermometer/USBCOMFlashTx.html){:target="_blank"} de `pvvx` y conectar con el dispositivo mediante un adaptador como este [**Hailege CP2102 Adaptador USB a TTL UART 232**](https://amzn.to/42chMqI){:target="_blank"} (8,99€):

![Hailege CP2102 Adaptador USB a TTL UART 232 con 485 puertos][2]

Tal como explica `pvvx` en la [documentación del USB-COM](https://github.com/pvvx/ATC_MiThermometer?tab=readme-ov-file#the-usb-com-adapter-writes-the-firmware-in-explorer-web-version){:target="_blank"} y el usuario `Tuti4120` en [este post de la _issue_ 298](https://github.com/atc1441/ATC_MiThermometer/issues/298#issuecomment-2559632638){:target="_blank"} y en [este otro post de la misma _issue_](https://github.com/atc1441/ATC_MiThermometer/issues/298#issuecomment-2561858712){:target="_blank"} únicamente hay que realizar estas conexiones antes de poder enviar el _firmware_ a través del adaptador USB a TTL:

* 3.3V &rarr; CR2032 Battery +
* GND  &rarr; CR2032 Battery -
* TXD  &rarr; P14

Además, según la [documentación del adaptador CP2102](https://m.media-amazon.com/images/I/A147pZXYInL.pdf){:target="_blank"}, el modo **USB to TTL** se configura mediante un _switch_ DIP de dos dígitos y un _switch_ SMD de la siguiente manera:

* Dialing 1 (USB): **On**
* Dialing 2 (485): **Off**
* Switch (S1): **Up** (232-TTL)

# Drivers CP210x

El adaptador [**Hailege CP2102 Adaptador USB a TTL UART 232**](https://amzn.to/42chMqI){:target="_blank"} incorpora el chip [CP2102 USB to UART Bridge](https://www.silabs.com/interface/usb-bridges/classic/device.cp2102?tab=specs){:target="_blank"} de Silicon Labs.

Al conectar por primera vez el adaptador a un ordenador con Windows (en mi caso he usado Windows Server 2022) puede que no tenga los controladores necesarios para funcionar tal como muestra el Administrador de Dispositivos:

![CP2102 USB to UART Bridge Controller][3]

Por ese motivo, es necesario descargar el [CP210x Universal Windows Driver](https://www.silabs.com/documents/public/software/CP210x_Universal_Windows_Driver.zip){:target="_blank"} (versión 11.4.0, 18 Diciembre 2024) y descomprimirlo en un directorio temporal donde veremos el siguiente contenido:

![CP210x Universal Windows Driver][4]

A continuación, únicamente hay que seleccionar el dispositivo y actualizar su controlador usando el contenido de esta carpeta. Si todo funciona correctamente, el dispositivo **Silicon Labs CP210x USB to UART Bridge** quedará configurado en un puerto `COMx`, listo para ser utilizado:

![Silicon Labs CP210x USB to UART Bridge (COM3)][5]

# Procedimiento de _flasheo_

Una vez tenemos todo preparado, ha llegado la hora de realizar el _flasheo_ de un `LYWSD03MMC` con el último _firmware_ disponible a día de hoy [`ATC_v49.bin`](https://github.com/pvvx/ATC_MiThermometer/raw/refs/heads/master/ATC_v49.bin){:target="_blank"} usando el adaptador USB-COM `CP2102`.

## Desmontar el termostato

Para ello, tenemos que abrir el termostato siguiendo estos pasos:

- Quitar con cuidado la tapa posterior deslizando una pequeña pieza de plástico por la ranura de la parte inferior
- Sacar la batería CR2032 del dispositivo
- Quitar los dos pequeños tornillos que sujetan la carcasa interior usando un **destornillador T4**
- Sacar con cuidado la carcasa interior usando un pequeño destornillador plano para hacer palanca en la parte superior
- Sacar el circuito del termostato (con cuidado para no sacar también el LCD ;-)

![Despiece LYWSD03MMC][6]

## Preparar las conexiones

A continuación debemos conectar [**3 cables Dupont Macho-Hembra**](https://www.mattmillman.com/info/crimpconnectors/dupont-and-dupont-connectors/){:target="_blank"} en los siguientes pines del adaptador CP2102:

* 3.3V &rarr; Cable rojo
* GND  &rarr; Cable negro
* TXD  &rarr; Cable azul

![Conexiones Dupont en CP2102][7]

El otro extremo de los cables Dupont rojo y negro se conectarán a los bornes de la batería CR2032 para alimentar el Xiaomi LYWSD03MMC:

* Cable rojo &rarr; Battery + (la parte lateral de la CR2032)
* Cable negro &rarr; Battery - (la parte central)

Los cables rojo y negro se pueden fijar fácilmente pasándolos por debajo de los bornes donde se coloca la batería CR2032.

![Conexiones Dupont en LYWSD03MMC][8]

## Conectar el USB-COM

A continuación se conecta el adaptador USB-COM al ordenador y se espera a que sea detectado en el Administrador de dispositivos (la luz naranja se enciende).

- Acceder a la herramienta web [https://pvvx.github.io/ATC_MiThermometer/USBCOMFlashTx.html](https://pvvx.github.io/ATC_MiThermometer/USBCOMFlashTx.html){:target="_blank"}
- Pulsar el botón `Open` y seleccionar el dispositivo `CP2102 USB to UART Bridge Controller (COM3) - Paired`
- Pulsar el botón `Firmware` y seleccionar el fichero `ATC_v49.bin` que habíamos descargado previamente
- Hacer contacto con el **cable azul (TXD) en el punto P14** y sujetarlo bien con la mano durante el _flasheo_ (hay que tener un buen pulso ;-).
- Pulsar el botón `Write to Flash` para que empiece el proceso (la luz azul del TX parpadeará)

Al cabo de unos 34 segundos finaliza el envío del _firmware_ que **se envía sin comprobación alguna**.

{: .box-note}
Si no estamos seguros de haber mantenido el cable azul en el punto P14 y se ha movido, es recomendable volver a repetir el proceso para asegurar que se envía completamente el _firmware_.

![TLSR825x USB-COM Flash Writer][9]

- Soltar el cable azul (P14) y desconectar los cables rojo y negro de los bornes de la batería

## Montaje del termostato

Finalmente, se procede a montar todas las piezas del termostato en orden inverso:

- Asegurarse que el LCD está plano y bien colocado en su hueco
- Colocar el circuito dentro de la caja alineando el agujero de la parte superior izquierda con un pequeño pin de plástico
- Colocar la carcasa interior apretando bien en las cuatro esquinas (si no está bien apretado no se verá bien la pantalla LCD)
- Colocar la batería CR2032 y comprobar que el termostato funcione y la pantalla funcione
- Acceder a la herramienta web [https://pvvx.github.io/ATC_MiThermometer/TelinkMiFlasher.html](https://pvvx.github.io/ATC_MiThermometer/TelinkMiFlasher.html){:target="_blank"}
- Conectar con el dispositivo `ATC_XXXXXX` para comprobar que se ha _flasheado_ correctamente el _firmware_ 4.9

Si todo ha salido según lo previso, se mostrará información sobre el _firmware_ y los datos de batería, temperatura, humedad, etc.

![Firmware 0.49 OK][10]

- Colocar los dos pequeños tornillos que sujetan la carcasa interior usando un destornillador T4
- Colocar con cuidado la tapa posterior

# Referencias

* [Xiaomi LYWSD03MMC Bluetooth to Zigbee from the "unsupported" firmware version 2.1.1.0159](https://www.youtube.com/watch?v=BtsxkOS6Zj8){:target="_blank"} @ YouTube: AI Home Tech Lab
* [Flashing of LYWSD03MMC B1.5](https://www.youtube.com/watch?v=ygJSTk1s7qU){:target="_blank"} @ YouTube: Raked T. (Tuti4120)

### Historial de cambios

* **2025-01-12**: Documento inicial

[1]: /assets/img/blog/2025-01-12_image_1.png "Firmware 2.1.1_0159"
[2]: /assets/img/blog/2025-01-12_image_2.png "Hailege CP2102 Adaptador USB a TTL UART 232 con 485 puertos"
[3]: /assets/img/blog/2025-01-12_image_3.png "CP2102 USB to UART Bridge Controller"
[4]: /assets/img/blog/2025-01-12_image_4.png "CP210x Universal Windows Driver"
[5]: /assets/img/blog/2025-01-12_image_5.png "Silicon Labs CP210x USB to UART Bridge (COM3)"
[6]: /assets/img/blog/2025-01-12_image_6.png "Despiece LYWSD03MMC"
[7]: /assets/img/blog/2025-01-12_image_7.png "Conexiones Dupont en CP2102"
[8]: /assets/img/blog/2025-01-12_image_8.png "Conexiones Dupont en LYWSD03MMC"
[9]: /assets/img/blog/2025-01-12_image_9.png "TLSR825x USB-COM Flash Writer"
[10]: /assets/img/blog/2025-01-12_image_10.png "Firmware 0.49 OK"