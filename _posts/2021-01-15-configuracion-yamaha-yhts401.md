---
layout : post
blog-width: true
title: 'Configuración Yamaha YHT-S401'
date: '2021-01-15 19:09:01'
published: true
tags:
- Hardware
author:
  display_name: Manel Rodero
---

En Febrero de 2012 compré un conjunto de _home theater_ **Yamaha YHT-S401** que aún conservo funcionando perfectamente. Este conjunto está formado por un receptor **Yamaha SR-301** y una barra de sonido **Yamaha NS-BR301** como las mostradas a continuación:

![Imagen][1]

# Configuración

La configuración que estoy usando en el equipo es la siguiente:

* 1:SP LEVEL
  * 1-1:FRONT L: +3
  * 1-2:FRONT R: +3
  * 1-3:CENTER: +4
  * 1-4:SURROUND L: +3
  * 1-5:SURROUND R: +3
  * 1-6:SUBWOOFER: +3

{: .box-note}
**Nota**: Si tuviera unos altavoces traseros instalados, el Surround L y R se podrían subir a +4. Dejándolo en +3 se "simulan" los altavoces traseros desde la propia barra.

* 2:TONE CONTROL
  * 2-1:BASS: +3
  * 2-2:TREBLE: +4

* 3:HDMI SETUP
  * 3-1:CONTROL: ON

{: .box-note}
**Nota**: El ARC (Audio Return Channel) está activado para poner en marcha y apagar el equipo junto con la televisión. Es recomendable desactivar el ARC en los dispositivos que no sean la televisión para evitar conflictos entre ellos.
 
* 4:DISPLAY MODE
  * DIMMER: -2

{: .box-note}
**Nota**: El _dimmer_ es el brillo del _display_ para que no moleste si la habitación está a oscuras.

* 5:SP SETUP
  * 5-1:SP CHANNEL: 3CH
  * 5-2:SP TYPE: BAR

{: .box-note}
**Nota**: Si tuviera unos altavoces traseros instalados, se usaría 5CH.

* 6:FIRMWARE
  * 6-1:VERSION: 6.03
  * 6-2:USB UPDATE

* 7:D. RANGE
  * D.DRC: STANDARD

{: .box-note}
**Nota**: Si tuviera unos altavoces traseros instalados (es decir 5CH), se usaría Standard o Max.
 
# Opciones

Las opciones de cada entrada se han configurado así:

* OPTION
  * 1.VOLUME TRIM: +1
  * 2.AUDIO DELAY: 0
  * 3.AUDIO ASSIGN: HDMI (únicamente para entradas HDMI)

El resto de opciones se activan desde el mando y suelo usarlas así:

  * CLEAR VOICE: Activado
  * SURROUND: MOVIE, SPORT, TV, etc. en función de lo que se esté viendo
  * ENHANCER: Desactivado
  * UNIVOLUME: Desactivado

{: .box-note}
**Nota**: La función _Enhancer_ sirve para mejorar la reprodución de música comprimida como MP3 desde el USB.

# Pruebas

Se han descargado diferentes ficheros para reproducirlos usando [OpenELEC](https://openelec.tv/) en una [Raspberry Pi 3 Model B+](https://www.raspberrypi.org/products/raspberry-pi-3-model-b-plus/):

* [High Resolution Music](http://www.2l.no/hires/index.html)
* [Demo World](https://www.demo-world.eu/movie-trailers-hd/)

Los formatos detectados por el equipo según el tipo de fichero:

| Tipo de fichero | Formato detectado |
| --- | --- |
| Dolby TrueHD | ][ TRUE HD |
| Dolby Digital Plus | ][ DIGITAL PLUS |
| Dolby Digital | |
| Dolby Pro Logic | |
| Dolby Pro Logic II | |
| DTS-HD Master Audio	| dts HD MSTR |
| DTS-HD High Resolution | dts HD HI RES |
| DTS	| dts |
| DTS 96/24 | |

<p></p>

[1]: /assets/img/blog/2021-01-15_image_1.png "Imagen"
