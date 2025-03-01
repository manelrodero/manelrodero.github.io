---
layout: post
blog-width: true
title: Edición de fotografías JPEG sin pérdida de calidad
date: 2006-03-16 23:15:56
last-updated: '2025-03-01 18:36:11'
published: true
tags:
- Fotografía
author:
  display_name: Manel Rodero
---

La edición de una fotografía directamente en formato JPEG implica una pérdida de calidad importante cada vez que se graba el fichero a disco.

Esto es debido a que muchos editores de imágenes decodifican la imagen JPEG al leerla por primera vez, hacen las modificaciones indicadas por el usuario sobre el mapa de bits de bits y, finalmente, vuelven a codificar la imagen en formato JPEG antes de escribirla de nuevo en el disco.

Esta codificación en formato JPEG implica un cierto nivel de compresión (elegido por el usuario con un compromiso entre tamaño del fichero y calidad de la imagen) y, por tanto, una pérdida de información (en inglés loss o additive compression error).

Por esta razón es recomendable usar formato de fichero sin pérdida (lossless) como TIFF, PSD, BMP, PSP, etc. durante el proceso de edición. JPEG sólo se debería usar para guardar la imagen final y posiblemente para la captura inicial.

Si no tenemos en cuenta esta pérdida del formato JPEG, operaciones tan sencillas como la rotación de una fotografía o un recorte de la misma, pueden significar una pérdida de calidad que redundará negativamente a la hora de imprimir la fotografía en papel o, según los parámetros de compresión JPEG, incluso a la hora de visualizarla en pantalla.

Más información en:

* [JPEG Compression, Quality and File Size](https://web.archive.org/web/20060318120945/http://www.impulseadventure.com/photo/jpeg-compression.html){:target="_blank"}
* [JPEG Lossless Rotation, Lossless Crop](https://web.archive.org/web/20060318120945/http://www.impulseadventure.com/photo/lossless-rotation.html){:target="_blank"}
* [Lossless Rotation, lossless crop - How to test?](https://web.archive.org/web/20060318120945/http://www.impulseadventure.com/photo/lossless-rotation-test.html){:target="_blank"}

Software útil para evitar estos problemas:

* JPEG Lossless Rotator - Freeware
* [Better JPEG](http://www.betterjpeg.com/index.htm){:target="_blank"} - $24.95
* [Better JPEG Lossless Resave Plugin for Adobe Photoshop](http://www.betterjpeg.com/jpeg-plug-in.htm){:target="_blank"} - $24.95
* [ionForge imageDiff](https://web.archive.org/web/20060208091838/https://www.ionforge.com/products/imagediff/){:target="_blank"} - Freeware
* [jpgQ - JPEG Quality Estimator](http://www.mediachance.com/digicam/jpgq.htm){:target="_blank"} - Freeware

### Historial de cambios

* **2006-03-16**: Documento inicial
* **2025-02-28**: Corrección de enlaces (WebArchive.org)
