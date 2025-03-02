---
layout: post
blog-width: true
title: "¿Cómo comprobar la integridad de ficheros de vídeo?"
date: 2012-09-18 21:49:09
published: true
tags:
- Vídeo
author:
  display_name: Manel Rodero
---

Hace unos días, un compañero de trabajo me pidió unos capítulos de una serie que quería ver. Le copié en un disco duro USB un par de temporadas. Hoy me ha devuelto el disco duro para que le pase los capítulos restantes y me ha comentado que alguno de los vídeos estaba mal y no se veía correctamente. Lo he comprobado usando **MPC-HC** y, efectivamente, el vídeo se ve como en la siguiente captura:

![MKV Corrupto][1]

Como recordaba que me había comentado que en las primeras temporadas también había algún vídeo que estaba mal, he buscado un método para poder comprobar todos los vídeos sin tener que mirarlos uno a uno. Al final la solución ha sido la herramienta de conversión universal: [**FFmpeg**][2]{:target="_blank"}.

La idea fundamental es "convertir" el vídeo, leyendo el fichero completo y enviando la salida al dispositivo nulo. Como no hay una conversión real del vídeo, el proceso es bastante rápido. El comando para realizar esta comprobación es el siguiente:

```plaintext
ffmpeg -v error -i "Video.mkv" -f null -  
```

Si el vídeo es correcto, el comando anterior no muestra ninguna salida. Si el vídeo tiene algún tipo de error, lo indicará por pantalla con mensajes similares al siguiente:

```plaintext
[h264 @ 03d77740] error while decoding MB 7 15, bytestream (-5) [matroska,webm @ 01eeae20]  
Read error at pos. 599264369 (0x23b80c71) [matroska,webm @ 01eeae20] Read error at pos. 599264370 (0x23b80c72)  
```

¡Ha llegado el momento de comprobar la integridad del resto de ficheros!

[1]: /assets/img/blog/2012-09-18_image_1.jpg
[2]: https://ffmpeg.org/
