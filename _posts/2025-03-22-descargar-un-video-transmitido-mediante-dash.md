---
layout : post
blog-width: true
title: 'Descargar un vídeo transmitido mediante DASH'
date: '2025-03-22 14:10:10'
#last-updated: '2025-03-22 14:10:10'
published: true
tags:
- PowerShell
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2025-03-22_cover.png"
thumbnail-img: ""
---

Cuando tengo que descargar algún vídeo suelo utilizar [**yt-dlp**](https://github.com/yt-dlp/yt-dlp){:target="_blank"}, una herramienta de línea de comandos diseñada para descargar videos y audios desde una amplia variedad de sitios web.

`yt-dlp` es un _fork_ de la popular herramienta `youtube-dl`, pero incluye muchas características adicionales y mejoras. Por ejemplo, es compatible con protocolos de transmisión adaptativa como **DASH** y **HLS**, lo que permite descargar segmentos y reconstruirlos en un archivo completo.

Ahora bien, cuando la página web no es una de las soportadas por el proyecto o no se consigue detectar el _stream_ de vídeo correctamente, lo más sencillo es intentar buscarlo mediante las [**Developer Tools**](https://learn.microsoft.com/en-us/microsoft-edge/devtools-guide-chromium/overview){:target="_blank"} del navegador.

# DASH y HLS

[**DASH (Dynamic Adaptive Streaming over HTTP)**](https://en.wikipedia.org/wiki/Dynamic_Adaptive_Streaming_over_HTTP){:target="_blank"} es un protocolo de transmisión que divide un video o audio en múltiples segmentos pequeños que se descargan y reproducen de manera dinámica.

Permite ajustar la calidad de reproducción en tiempo real dependiendo del ancho de banda disponible, garantizando una experiencia sin interrupciones. Se basa en ficheros de control (como un manifiesto o `playlist.json` en este caso) que indican cómo acceder a los segmentos de video y audio.

[**HLS (HTTP Live Streaming)**](https://en.wikipedia.org/wiki/HTTP_Live_Streaming){:target="_blank"} es otro protocolo de transmisión adaptativa, desarrollado por Apple.

Similar a DASH, divide el contenido en pequeños fragmentos y usa listas de reproducción (`.m3u8`) para coordinar la reproducción. Se utiliza ampliamente en dispositivos y servicios compatibles con Apple, pero también tiene soporte en otras plataformas.

Ambos protocolos son fundamentales para la transmisión en _streaming_ porque permiten entregar contenido de alta calidad de manera eficiente, independientemente de las condiciones de la red del usuario.

# `playlist.json`

En el caso que nos ocupa, después de abrir las herramientas de desarrollo usando **`F12`**, se pueden filtrar las peticiones XHR realizadas por el navegador cuando se reproduce el vídeo para encontrar el fichero `playlist.json`:

![playlist.json][1]

{: .box-note}

[**XHR (XMLHttpRequest)**](https://en.wikipedia.org/wiki/XMLHttpRequest){:target="_blank"} es un mecanismo en los navegadores que permite realizar solicitudes HTTP (GET, POST, etc.) desde JavaScript. Se usa para obtener datos (como el fichero JSON en nuestro caso) desde un servidor sin necesidad de recargar la página.

## Estructura del fichero JSON

La estructura de este fichero JSON, simplificada bastante para dejar únicamente un bloque de vídeo y uno de audio, es la siguiente:

```json
{
  "clip_id": "<guid_del_clip>",
  "base_url": "../url/base/del/video/",
  "video": [
    {
      "id": "<guid_del_video>",
      "format": "dash",
      "mime_type": "video/mp4",
      "codecs": "avc1.64001F",
      "bitrate": 1254000,
      "avg_bitrate": 808000,
      "duration": 159.92,
      "framerate": 25,
      "width": 960,
      "height": 540,
      "init_segment": "<segmento_init_en_Base64>",
      "index_segment": "<guid_del_segmento_index>.mp4?<parámetros>&range=806-1161",
      "segments": [
        {
          "url": "<guid_del_segmento_1>.mp4?<parámetros>&range=1162-312730",
          "size": 311568
        },
        {
          "url": "<guid_del_segmento_n>.mp4?<parámetros>&range=15637804-15996276",
          "size": 358472
        }
      ]
    }
  ],
  "audio": [
    {
      "id": "<guid_del_audio>",
      "format": "dash",
      "mime_type": "audio/mp4",
      "codecs": "mp4a.40.2",
      "bitrate": 195000,
      "avg_bitrate": 194000,
      "duration": 159.936,
      "channels": 2,
      "sample_rate": 48000,
      "init_segment": "<segmento_init_en_Base64>",
      "init_segment_url": "",
      "index_segment": "<guid_del_segmento_index>.mp4?<parámetros>&range=678-1033",
      "segments": [
        {
          "url": "<guid_del_segmento_1>.mp4?<parámetros>&range=1034-146462",
          "size": 145428
        },
        {
          "url": "<guid_del_segmento_n>.mp4?<parámetros>&range=3780771-3880609",
          "size": 99838
        }
      ],
      "audio_primary": true
    }
  ]
}
```

## Propiedades del fichero JSON

Las **propiedades principales** de este fichero son `video` y `audio`, las cuales contienen las listas de configuraciones de video y audio disponibles para ser reproducidas.

En cada **bloque de video o audio** (puede haber más de uno de cada) se especifican las características del mismo:

- `format`: El formato del bloque (por ejemplo, `dash`)
- `mime_type`: El tipo MIME, que describe el tipo de archivo (por ejemplo, `video/mp4` o `audio/mp4`)
- `codecs`: El codec utilizado para codificar el bloque (por ejemplo, `avc1` o `mp4a`)
- `bitrate`: La tasa de bits del bloque, que afecta la calidad
- `segments`: Una lista de fragmentos que forman ese bloque

Cada **segmento** de vidoe o audio tiene, como mínimo, los siguientes valores:

- `url`: La dirección específica para descargar ese fragmento
- `size`: El tamaño del fragmento, útil para validaciones

### `init_segment`

Además, en el caso de este JSON concreto, se definen un **`init_segment`** y un **`index_segment`** que juegan un papel importante dentro de **DASH** y otros protocolos de transmisión adaptativa.

El `init_segment` (segmento de inicialización) contiene los **metadatos** necesarios para comenzar la reproducción de las pistas de video o audio. Estos metadatos incluyen información como el formato del archivo, las cabeceras del contenedor (por ejemplo, MP4), los detalles del codec, y otra información esencial para decodificar y reproducir los segmentos posteriores.

Este segmento **no contiene datos de video o audio directamente reproducibles**, pero es obligatorio para que el reproductor pueda interpretar los bloques posteriores.

En algunos ficheros JSON (como el que analizamos), el `init_segment` puede estar directamente embebido en el JSON como una cadena codificada en [**Base64**](https://en.wikipedia.org/wiki/Base64){:target="_blank"}. Esto permite que el cliente (por ejemplo, un reproductor o un script) lo decodifique y utilice sin necesidad de descargar un archivo separado.

La codificación Base64 transforma los datos binarios en texto que es seguro para transmitir en JSON, ya que no incluye caracteres que puedan romper el formato o ser malinterpretados, por ejemplo:

```json
"init_segment": "AAAAHGZ0eXBkYXNoAAAAAGRhc2htcDQyaXNvNg..."
```

En este caso, sería necesario decodificar el contenido para generar un archivo binario válido. Solo se descarga una vez por pista (video o audio) y se utiliza como base para interpretar todos los fragmentos (o `segments`) de esa pista.

### `index_segment`

El `index_segment` (segmento de índice) proporciona información adicional sobre cómo están estructurados y ordenados los `segments`. Puede contener datos sobre las ubicaciones byte a byte de los segmentos dentro de la pista o servir como referencia para acceder a los fragmentos específicos.

En muchos casos, el `index_segment` no es necesario descargarlo explícitamente, ya que los reproductores pueden inferir su contenido o acceder directamente a las URLs de los `segments`.

Aunque en algunos sistemas de transmisión adaptativa el `index_segment` es un archivo real, en otros, se trata de información implícita que el reproductor utiliza al leer el manifiesto (como en el JSON que estás trabajando).

# Script en PowerShell

Una vez conocemos los detalles del protocolo DASH y el contenido del fichero `playlist.json`, se puede construir un _script_ en PowerShell que lo procese de la siguiente manera:

1. **Obtención del JSON**
   - Descargar el fichero JSON inicial que contiene información de los bloques de video y audio

![Obtener JSON][2]

2. **Selección dinámica**
   - Permitir al usuario elegir qué configuración de video y audio desea descargar (ordenadas y presentadas de manera clara)

![Selección dinámica del vídeo][3]

3. **Descarga y concatenación de segmentos**
   - Los segmentos de video y audio seleccionados se descargan pieza por pieza usando **`curl.exe`**
   - Se valida cada descarga para garantizar que los archivos sean correctos
   - Los segmentos descargados se combinan de manera incremental (usando `cmd.exe /c copy /b`) para formar un único archivo de video o audio

![Descarga de segmentos del vídeo][4]

4. **Combinación de video y audio**
   - Al final, se utiliza [**FFmpeg**](https://ffmpeg.org/){:target="_blank"} para fusionar el archivo de video y audio en un único archivo reproducible

![Combinación de vídeo y audio][5]

{: .box-warning}
No proporciono el código fuente del _script_, más allá de las capturas anteriores, porque cada web requiere una programación diferente. Ahora tienes las bases para hacerlo tú mismo y [Microsoft Copilot](https://copilot.microsoft.com/){:target="_blank"} es tu amigo ;-)

## Ejemplo de ejecución

Una vez se ha creado y probado el _script_ en PowerShell, llega la hora de ejecutarlo:

```plaintext
PS C:\Users\Test> .\Download-DASH-Video-v10.ps1

cmdlet Download-DASH-Video-v10.ps1 at command pipeline position 1
Supply values for the following parameters:
Url: https://web-video.com/videos/34f1eb03-7655-4ab8-cd25-a4abad796c92/playlist.json
Title: Test Video

Bloques de Video disponibles:
Número Formato MIME_Type Codecs      Bitrate Avg_Bitrate Framerate Ancho Alto
------ ------- --------- ------      ------- ----------- --------- ----- ----
     5 dash    video/mp4 avc1.64002A 5418000     3970000        25  1920 1080
     1 dash    video/mp4 avc1.640020 2673000     1753000        25  1280  720
     2 dash    video/mp4 avc1.64001F 1475000     1062000        25   960  540
     3 dash    video/mp4 avc1.64001E  745000      554000        25   640  360
     4 dash    video/mp4 avc1.640015  328000      276000        25   426  240

Introduce el número del bloque de video que deseas descargar: 5

Bloques de Audio disponibles:
Número Formato MIME_Type Codecs    Bitrate Avg_Bitrate Canales Frecuencia
------ ------- --------- ------    ------- ----------- ------- ----------
     1 dash    audio/mp4 mp4a.40.2  196000      194000       2      48000
     3 dash    audio/mp4 opus        70000       68000       2      48000
     2 dash    audio/mp4 opus       102000       99000       2      48000

Introduce el número del bloque de audio que deseas descargar: 1

init_segment de video descargado y configurado como base: C:\Users\Test\Video\video_final.mp4
Segmento 1 descargado correctamente: C:\Users\Test\Video\segmento_video_1.mp4 (Tamaño: 1835244)
Segmento de video concatenado: C:\Users\Test\Video\video_final.mp4
[...]
Segmento 134 descargado correctamente: C:\Users\Test\Video\segmento_video_134.mp4 (Tamaño: 2009319)
Segmento de video concatenado: C:\Users\Test\Video\video_final.mp4

init_segment de audio descargado y configurado como base: C:\Users\Test\Audio\audio_final.mp4
Segmento 1 descargado correctamente: C:\Users\Test\Audio\segmento_audio_1.mp4 (Tamaño: 145680)
Segmento de audio concatenado: C:\Users\Test\Audio\audio_final.mp4
[...]
Segmento 134 descargado correctamente: C:\Users\Test\Audio\segmento_audio_134.mp4 (Tamaño: 137496)
Segmento de audio concatenado: C:\Users\Test\Audio\audio_final.mp4

Video y audio combinados exitosamente en: C:\Users\Test\Test Video.mp4
```

![Test Video.mp4][6]

### Historial de cambios

- **2025-03-22**: Documento inicial

[1]: /assets/img/blog/2025-03-22_image_1.png "playlist.json"
[2]: /assets/img/blog/2025-03-22_image_2.png "Obtener JSON"
[3]: /assets/img/blog/2025-03-22_image_3.png "Selección dinámica del vídeo"
[4]: /assets/img/blog/2025-03-22_image_4.png "Descarga de segmentos del vídeo"
[5]: /assets/img/blog/2025-03-22_image_5.png "Combinación de vídeo y audio"
[6]: /assets/img/blog/2025-03-22_image_6.png "Test Video.mp4"