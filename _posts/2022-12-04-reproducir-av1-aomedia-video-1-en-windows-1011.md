---
layout : post
blog-width: true
title: 'Reproducir AV1 (AOMedia Video 1) en Windows 10/11'
date: '2022-12-04 21:03:43'
published: true
tags:
- Windows
author:
  display_name: Manel Rodero
---

Hoy me han pasado un vídeo que Windows Media Player en Windows 11 no podía visualizar.

Usando [MediaInfo](https://mediaarea.net/en/MediaInfo) he obtenido información sobre el códec que usaba y se trataba de[AOMedia AV1](https://aomedia.org/av1-features/), una alternativa a H264 optimizado para _streaming_ y resoluciones hasta 4K UHD:

![Imagen][1]

Este nuevo códec tiene una mayor compresión (hasta un 30% más) que sus alternativas.

Por ejemplo, **1 hora** en AVC ocupa **3.72 GiB**:

```
Format                                   : AVC
Format/Info                              : Advanced Video Codec
Format profile                           : High@L4
Codec ID                                 : V_MPEG4/ISO/AVC
Duration                                 : 1 h 1 min
Bit rate                                 : 8 648 kb/s
Width                                    : 1 920 pixels
Height                                   : 1 080 pixels
Frame rate mode                          : Constant
Frame rate                               : 23.976 FPS
Stream size                              : 3.72 GiB (100%)
```

En cambio, en AV1 **53 minutos** ocupan únicamente **304 MiB**:

```
Format                                   : AV1
Format/Info                              : AOMedia Video 1
Format profile                           : Main@L4.0
Codec ID                                 : V_AV1
Duration                                 : 52 min 44 s
Bit rate                                 : 805 kb/s
Width                                    : 1 920 pixels
Height                                   : 1 080 pixels
Frame rate mode                          : Constant
Frame rate                               : 23.976 (24000/1001) FPS
Stream size                              : 304 MiB (100%)
```

Como este códec está llamado a sustituir al `VP9` de Google o al costoso `HEVC/H.265`, es interesante poder añadir soporte a Windows 11 de la siguiente manera:

* Abrir la **Microsoft Store**
* Buscar [`AV1 Video Extension`](https://www.microsoft.com/store/productId/9MVZQVXJBQ9V)
* Instalarlo

Una vez instalado, ya se podrá reproducir cualquier fichero en formato `AV1` que nos encontremos.

[1]: /assets/img/blog/2022-12-04_image_1.png "Imagen"
