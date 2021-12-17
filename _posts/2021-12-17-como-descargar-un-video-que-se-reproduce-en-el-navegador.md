---
layout : post
blog-width: true
title: '¿Cómo descargar un vídeo que se reproduce en el navegador?'
date: '2021-12-17 11:38:01'
published: true
tags:
- Internet
author:
  display_name: Manel Rodero
---

¿En alguna ocasión has querido descargar un vídeo que se está reproduciendo en el navegador pero no existe una opción para hacerlo desde la plataforma de reproducción?

Aunque existen programas que pueden grabar un _stream_ de audio y vídeo desde las plataformas más comunes (por ejemplo [youtube-dl](https://yt-dl.org/), [JDownloader](https://jdownloader.org/) o [Free Download Manager](https://www.freedownloadmanager.org/)) muchas veces es complicado hacerlo desde plataformas menos comunes o de formación.

En estos casos, se pueden utilizar las **herramientas de desarrollo** del navegador (normalmente accesibles mediante la tecla `F12`) para ver si es posible obtener la URL del vídeo y descargarlo utilizando el reproductor **[VLC media player](https://www.videolan.org/)**.

Los pasos a seguir son los siguientes:

* Abrir las herramientas de desarrollo (`F12`)
* Seleccionar la opción para ver el **tráfico de red**
* Filtrar las URL que contengan `m3u`
* Recargar la página web (`F5`)

Si la plataforma utiliza listas de reproducción en **[formato M3U](https://www.adslzone.net/reportajes/software/archivos-m3u/)**, las peticiones se filtrarán y se observará algo similar a la siguiente captura de pantalla:

![Captura de URLs M3U][1]

En este ejemplo se puede ver lo siguiente:

* Se ha descargado un primer fichero de texto con la lista de reproducción (`v2`)
* A continuació se descargan ficheros binarios con los diferentes fragmentos del vídeo (`seg-1-v1-a1.ts`, etc.)

Pulsando en el primer fichero, se obtiene una vista previa de su contenido tal como se puede ver en la siguiente captura de pantalla:

![Vista previa de fichero M3U][2]

El contenido es una lista de reproducción formada por las URL de los diferentes fragmentos del vídeo. El navegador los irá descargando en orden para que el visor de la plataforma los reproduzca:

```
#EXTM3U
#EXT-X-TARGETDURATION:3
#EXT-X-ALLOW-CACHE:YES
#EXT-X-PLAYLIST-TYPE:VOD
#EXT-X-VERSION:3
#EXT-X-MEDIA-SEQUENCE:1
#EXTINF:3.000,
/deliveries/97bad361c0cee335f06496a0f433d73c665a9903.m3u8/v2/seg-1-v1-a1.ts
#EXTINF:3.000,
/deliveries/97bad361c0cee335f06496a0f433d73c665a9903.m3u8/v2/seg-2-v1-a1.ts
#EXTINF:3.000,
/deliveries/97bad361c0cee335f06496a0f433d73c665a9903.m3u8/v2/seg-3-v1-a1.ts
...
```

Pulsando con el botón derecho encima de este fichero, aparece un menú contextual que permitirá **copiar la URL del fichero** y usarla para abrir el _stream_ en VLC media player:

![Obtención de la URL][3]

Una vez obtenida la URL, se puede utilizar `vlc.exe` para descargar el vídeo en un fichero `video.mp4`:

```
:: Descargar video
vlc.exe "https://<website>/deliveries/97bad361c0cee335f06496a0f433d73c665a9903.m3u8/v2" --sout file:video.mp4 --play-and-exit
```

# Vídeo y audio por separado

En algunas plataformas el vídeo y el audio pueden estar en ficheros separados para poder tener diferentes resoluciones, códecs de audio, etc.

En ese caso, el _player_ de la página web se encarga de sincronizarlos durante la reproducción. Por ejemplo, en la siguiente captura de pantalla se puede ver uno de estos casos:

![Vídeo y audio por separado][4]

El contenido del primer fichero `manifest(format...` que contiene la lista de formatos es el siguiente:

```
#EXTM3U
#EXT-X-VERSION:4
#EXT-X-MEDIA:TYPE=AUDIO,GROUP-ID="audio",NAME="aac_UND_2_127",DEFAULT=YES,URI="QualityLevels(127999)/Manifest(aac_UND_2_127,format=m3u8-aapl)"
#EXT-X-STREAM-INF:BANDWIDTH=748797,RESOLUTION=1280x720,CODECS="avc1.42c01f,mp4a.40.2",AUDIO="audio"
QualityLevels(588712)/Manifest(video,format=m3u8-aapl)
#EXT-X-I-FRAME-STREAM-INF:BANDWIDTH=748797,RESOLUTION=1280x720,CODECS="avc1.42c01f",URI="QualityLevels(588712)/Manifest(video,format=m3u8-aapl,type=keyframes)"
#EXT-X-STREAM-INF:BANDWIDTH=1502416,RESOLUTION=1920x1080,CODECS="avc1.42c028,mp4a.40.2",AUDIO="audio"
QualityLevels(1326108)/Manifest(video,format=m3u8-aapl)
#EXT-X-I-FRAME-STREAM-INF:BANDWIDTH=1502416,RESOLUTION=1920x1080,CODECS="avc1.42c028",URI="QualityLevels(1326108)/Manifest(video,format=m3u8-aapl,type=keyframes)"
```

Como se puede observar, en este fichero se indican diferentes resoluciones (1280x720, 1920x1080) y códecs de audio (avc1, mp4a) que se podrán escoger desde el _player_ de la página web.

El siguiente fichero `Manifest(video...` contiene la lista de reproducción con los fragmentos de vídeo:

![Fragmentos de vídeo][5]

Y, finalmente, el fichero `Manifest(aac_UND...` contiene la lista de reproducción con los fragmentos de audio:

![Fragmentos de audio][6]

Para poder **descargar** el vídeo y el audio y **juntarlos** de forma sincronizada, se necesitará copiar las URL de estos dos ficheros y utilizar `vlc.exe` de la siguiente manera:

```
:: Descargar video
vlc.exe "https://<website>/686b6469-9e5d-4f01-81a0-9748c490b310/595cc7d7f5f4b128dcbc5c28.ism/QualityLevels(588712)/Manifest(video,format=m3u8-aapl)" --sout file:video-only.mp4 --play-and-exit

:: Descargar audio
vlc.exe "https://<website>/686b6469-9e5d-4f01-81a0-9748c490b310/595cc7d7f5f4b128dcbc5c28.ism/QualityLevels(127999)/Manifest(aac_UND_2_127,format=m3u8-aapl)" --sout file:audio.mp4 --play-and-exit

:: Juntar video + audio
vlc.exe video-only.mp4 --input-slave=audio.mp4 --sout file:video.mp4 --play-and-exit

:: Borrar temporales
del video-only.mp4
del audio.mp4
```

¡Espero que esta información os pueda ser de utilidad en algún momento!


[1]: /assets/img/blog/2021-12-17_image_1.png "Captura de URLs M3U"
[2]: /assets/img/blog/2021-12-17_image_2.png "Vista previa de fichero M3U"
[3]: /assets/img/blog/2021-12-17_image_3.png "Obtención de la URL"
[4]: /assets/img/blog/2021-12-17_image_4.png "Vídeo y audio por separado"
[5]: /assets/img/blog/2021-12-17_image_5.png "Fragmentos de vídeo"
[6]: /assets/img/blog/2021-12-17_image_6.png "Fragmentos de audio"
