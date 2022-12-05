---
layout : post
blog-width: true
title: 'Reproducir HEVC (High Efficiency Video Coding) en Windows 10/11'
date: '2022-12-05 20:19:35'
published: true
tags:
- Windows
author:
  display_name: Manel Rodero
---

Hoy me han pasado un vídeo que Windows Media Player en Windows 11 no podía visualizar.

Usando [MediaInfo](https://mediaarea.net/en/MediaInfo) he obtenido información sobre el códec que usaba y se trataba de [HEVC](https://en.wikipedia.org/wiki/High_Efficiency_Video_Coding), una alternativa a H264 optimizado para _streaming_ y resoluciones hasta 4K UHD:

![H264 vs HEVC][1]

Este nuevo códec, también llamado **H.265**, comprime los vídeos de manera más eficiente que AVC/H.264.

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

En cambio, en HEVC **58 minutos** ocupan únicamente **304 MiB**:

```
Format                                   : HEVC
Format/Info                              : High Efficiency Format profile                           : Main 10@L4@Main
Codec ID                                 : hvc1
Duration                                 : 58 min 31 s
Bit rate                                 : 3 538 kb/s
Width                                    : 1 920 pixels
Height                                   : 1 080 pixels
Frame rate mode                          : Constant
Frame rate                               : 25.000 FPS
Stream size                              : 1.45 GiB (100%)
```

Aunque HEVC (H.265) puede comprimir videos el doble que AVC (H.264) con el mismo nivel de calidad, su principal problema es la reproducción sin decodificación acelerada por hardware.

Esto no es un problema para los reproductores de Blu-ray 4K (incluído el de la Xbox One) que se fabrican para soportarlo de forma nativa.

Para reproducir sin problemas en un ordenador es necesario contar, como mínimo, con el siguiente hardware:

* Intel 6a generación "Skylake" o CPU más nuevas
* AMD 6a generación "Carizzo" o APU más nuevas
* NVIDIA GeForce GTX 950, GTX 960 o o tarjetas gráficas más nuevas
* AMD Radeon R9 Fury, R9 Fury X, R9 Nano o tarjetas gráficas más nuevas

Es interesante poder añadir soporte para HEVC a Windows 11 de la siguiente manera:

* Abrir la **Microsoft Store**
* Buscar [`HEVC Video Extensions from Device Manufacturer`](https://www.microsoft.com/store/productId/9N4WGH0Z6VHQ)
* Instalarlo

![Microsoft Store (Free)][3]

> **Nota**: Según la búsqueda que se haga, o si Microsoft decide eliminar la versión anterior, es probable que se acabe encontrando la versión de pago del códec (0,99€):

![Microsoft Store (Pago)][2]

Una vez instalado, ya se podrá reproducir cualquier fichero en formato `HEVC` que nos encontremos.

# Referencias

* [What Is HEVC H.265 Video, and Why Is It So Important for 4K Movies?](https://www.howtogeek.com/342416/what-is-hevc-h.265-video-and-why-is-it-so-important-for-4k-movies/)
* [H.265 (HEVC) vs H.264 (AVC) Compression: Explained!](https://www.youtube.com/watch?v=Fawcboio6g4) @ YouTube/
HandyAndy Tech Tips

[1]: /assets/img/blog/2022-12-05_image_1.png "H264 vs HEVC"
[2]: /assets/img/blog/2022-12-05_image_2.png "Microsoft Store"
[3]: /assets/img/blog/2022-12-05_image_3.png "Microsoft Store"
