---
layout : post
blog-width: true
title: 'Mini PC Dell OptiPlex 7050 Micro'
date: '2023-07-08 10:09:00'
published: true
tags:
- HomeLab
author:
  display_name: Manel Rodero
---

# Infocomputer

El pasado 26 de junio compré, a través de [**Infocomputer**](https://www.info-computer.com/), un mini ordenador [Dell OptiPlex 7050 Micro](https://www.dell.com/support/home/es-es/product-support/product/optiplex-7050-micro/overview) **reacondicionado**.

Era mi primera compra en esta tienda y la atención telefónica funcionó a la perfección, solucionando todas mis dudas sobre el origen de los ordenadores, su tres años de garantía, la forma de pago, etc.

Elegí una unidad que tenía un procesador i5-7500T, 16GB de RAM DDR4 y un disco SSD de 256GB. Lo pagué mediante PayPal y, 24 horas más tarde, el 28 de junio ya lo tenía en casa. ¡Estoy feliz!

El ordenador llegó perfectamente embalado, pero al sacarlo de la caja comprobé que la pintura de la carcasa aún manchaba, ¡ups!. Al abrirlo para colocar un segundo disco SSD SATA, comprobé que el interior también tenía restos de la pintura en _spray_ y la limpieza dejaba mucho que desear porque el ventilador estaba lleno de polvo. ¡Me empiezo a preocupar!

Al ponerlo en marcha y realizar algunas comprobaciones con el Windows 10 que venía instalado, noté que el ordenador iba bastante lento y le faltaba fluidez en la interfaz gráfica. ¡Ahora ya estoy enfadado!

Comprobé que **la CPU no subía más de 0,79GHz** lo cual, según un artículo de Dell, podría ser causado por un [problema con el reloj RTC](https://www.dell.com/support/kbdoc/es-es/000145125/processor-speed-limited-to-0-79ghz-800mhz). Después de reiniciarlo, quitando la batería CMOS y pulsando 15 segundos el botón de encendido, el problema continuaba.

La actualización de la BIOS de la versión 1.4.4 con la que había llegado el ordenador a la versión **1.25.0** que descargué desde la página de soporte de Dell tampoco resolvió el problema. En este momento comencé a preguntarme por qué no se había comprobado el ordenador antes de enviarlo (BIOS, controladores, pruebas de rendimiento, etc.).

Finalmente, encontré un artículo de Johan Arwidmark donde explicaba que él tuvo el mismo problema con el mismo modelo de ordenador y fue debido a la [incompatibilidad del cargador AC](https://www.deploymentresearch.com/a-tale-of-the-slow-cpu-dell-optiplex-7050-running-on-0-79-ghz/) que, en mi caso, era de 120W.

Por tanto, el mismo día 28 de junio abrí una incidencia a través de la web de Infocomputer, explicando todo lo que había hecho, enviando fotografías del ordenador y un vídeo demostrativo de los problemas. Al poco rato me contestaron que enviase la unidad de vuelta y la cambiarían por otra que funcionase correctamente. Únicamente debía empaquetarla de nuevo y colocar una etiqueta en lugar visible con el texto `RETORNO+NUM.PEDIDO`. La empresa de mensajería [GLS](https://www.gls-spain.es/es/) debería ponerse en contacto conmigo para recogerlo en 24/48h.

El día 3 de julio aún no había pasado la empresa GLS a recoger el paquete. Después de varias llamadas a Infocomputer averigüé que tenían un nota de GLS indicando que "_habían pasado pero yo no estaba en casa_" (en realidad sí que estaba pero no tenía ninguna llamada en el videoportero digital). Al final, pedí el número de recogida y me planté en una oficina de GLS para entregarlo personalmente. ¿Tendría otro de vuelta rápidamente?

El día 5 de julio me llamó una persona del Servicio Técnico de Infocomputer indicando que "_había comprobado la unidad y, efectivamente, estaba defectuosa_". Al parecer el problema con el cargador era debido a la placa base y me pidió disculpas por ello. También me comentó que había revisado personalmente una nueva unidad y que me la enviaría esa misma tarde. ¡Gracias Yuri por tu ayuda!

Pero al día siguiente, después de esperar todo el día, no llegó el ordenador (¡snif, snif!) y tuve que llamar de nuevo el Servicio Técnico el día 7 de julio. El mismo técnico que me había atendido hacía dos días comprobó que mi paquete, y otros dos más que también eran "urgentes", se habían traspapelado en el almacén y nadie se había dado cuenta. Procedió a reclamar la recogida a GLS e indicó que se intentase entregar al día siguiente. ¡Gracias Yuri de nuevo por tu ayuda!

Finalmente, el sábado 8 de julio me llegó el ordenador perfectamente embalado, con una caja impecable sin pintura y completamente limpio en su interior. Según pude comrobar mediante el _service tag_ en la página de soporte de Dell, era una unidad diferente del mismo modelo pero con una antigüedad similar. Los componentes externos (CPU, memoria y disco) eran los iniciales y tenía un cargador de 130W. Lo puse en marcha y la velocidad de la CPU oscilaba según la carga tal como debe ser.

¡Por fín podía comenzar con la instalación del servidor!

# Dell OptiPlex 7050 Micro

## Hardware

Utilizando el **código de servicio rápido** de la unidad que me han enviado en la web de [Asistencia de Dell](https://www.dell.com/support/home/es-es), he podido consultar la antigüedad y características del ordenador:

* Proviene de Estados Unidos
* Tuvo una garantía de servicio desde el 16 de febrero de 2018 hasta el 17 de mayo de 2023
* Se montó con los siguientes componentes:
  * Intel Core i5-6500 @ 3,20GHz (TDP Max 65W)
  * Intel HD Graphics 530
  * 8GB DDR4-2400 @ 1200MHz
  * 500GB HDD SATA 2,5" 7200rpm
  * Adaptador AC 120W

Utilizando los **programas** [CPU-Z](https://www.cpuid.com/softwares/cpu-z.html), [HWiNFO](https://www.hwinfo.com/) y [CrystalDiskInfo](https://crystalmark.info/en/software/crystaldiskinfo/) he obtenido las **características reales** del _hardware_ recibido:

* Intel Core i5-7500T @ 2,70GHz (TDP Max 35W)
* Intel HD Graphics 630
* 16GB DDR4-2400 @ 1200 MHz, SK Hynix HMA82GS6AFR8N-UH
* 256GB SSD SATA @ 6 Gb/s, Micron 1100
* Adaptador AC 130W

Además, he aprovechado la inspección visual del interior del ordenador para añadir un **segundo SSD para datos** con las siguientes características:

* 1000GB SSD SATA @ 6 Gb/s, [Crucial CT1000MX500SSD1](https://www.crucial.es/ssd/mx500/ct1000mx500ssd1) ([Amazon](https://amzn.to/3PU2j8f))

## Configuración de la BIOS

A diferencia de la primera unidad enviada, esta segunda unidad ya tiene [actualizada la BIOS a la versión 1.25.0](https://www.dell.com/support/home/es-es/drivers/driversdetails?driverid=vwgkn&oscode=wt64a&productcode=optiplex-7050-micro) del 12 de mayo de 2023. Por tanto, únicamente he tenido que acceder a ella y configurarla de la siguiente manera:

* Iniciar el ordenador y pulsar `F2` para acceder a la BIOS
* Pulsar el botón `Restore Settings`
* Seleccionar la opción `BIOS Defaults` y pulsar el botón `OK`
* Aceptar la advertencia para cargar la nueva configuración pulsando el botón `OK`
* Aceptar la advertencia para reiniciar pulsando el botón `OK`

* Pulsar `F2` para acceder a la BIOS de nuevo
* General
  * Boot Sequence
    * Boot List Option
      * (x) UEFI
  * Advanced Boot Options
    * [ ] Enable Legacy Option ROMs
* System Configuration
  * Integrated NIC
    * [x] Enable UEFI Network Stack
    * (x) Enabled
  * SATA Operation
    * (x) AHCI
  * SMART Reporting
    * [x] Enable SMART Reporting
* Security
  * TPM 1.2 Security
    * [x] TPM On
    * (x) Enabled
  * SMM Security Mitigation
    * [x] SMM Security Mitigation
* Secure Boot
  * Secure Boot Enable
    * (x) Disabled
  * Expert Key Management
    * [x] Enable Custom Mode
* Intel Software Guard Extensions
  * Intel SGX Eanble
    * (x) Enabled
* Power Management
  * AC Recovery
    * (x) Power On
  * Deep Sleep Control
    * (x) Disabled
  * Wake on LAN/WLAN
    * (x) LAN Only
* POST Behavior
  * Extend BIOS POST Time
    * (x) 5 seconds
  * Warnings and Errors
    * (x) Continue on Warnings and Errors
* Virtualization Support
  * Trusted Execution
    * [x] Trusted Execution
* Pulsar `Apply` para aplicar los cambios
* Pulsar `Exit` para reiniciar el ordenador

## Wake-On-LAN

Se ha comprobado que el ordenador se inicia correctamente mediante _Wake-On-LAN_ usando el comando [`wolcmd.exe` de Depicus](https://www.depicus.com/wake-on-lan/wake-on-lan-cmd):

```
wolcmd.exe AA:BB:CC:DD:EE:FF 192.168.1.190 255.255.255.0 7
```

Finalmente, también se ha comprobado que el ordenador se pone en marcha después de un corte de luz gracias a la configuracion `Power On` de la sección `AC Recovery` de la BIOS.
