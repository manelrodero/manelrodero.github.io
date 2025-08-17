---
layout : post
blog-width: true
title: 'Colecciones y carátulas en Jellyfin'
date: '2025-08-17 08:46:16'
#last-updated: '2025-08-17 08:46:16'
published: true
tags:
- Software
author:
  display_name: Manel Rodero
#cover-img: "/assets/img/blog/2025-08-17_cover.png"
thumbnail-img: ""
---

Al definir una librería de tipo **Movies** desde `Dashboard > Libraries`, se puede marcar la opción _Automatically add to collection_ lo cual permite agrupar las películas si hay como mínimo dos de la misma colección.

## Problema

El problema es que, a veces, Jellyfin no descarga correctamente los metadatos y las imágenes para esa colección:

![Coleccion sin imagen][1]

Y lo que es peor, puede **bloquearla por defecto**, lo que impide que se actualice sola.

## Solución

Para solucionar este problema, hay que hacer lo siguiente:

* **Desbloquear la colección manualmente**:
  * Acceder a la colección desde la interfaz web
  * Hacer clic en los tres puntos (`⋮`) > **Edit Metadata**
  * Desmarcar la opción "**Lock this item to prevent future changes**" si está activada
  * Hacer clic en el botón `Save`
* **Refrescar los metadatos**:
  * Una vez desbloqueada, volver a los tres puntos (`⋮`) > **Refresh Metadata**
  * Seleccionar **Replace all metadata** y marcar:
    * ☑ Replace existing images
    * ☑ Replace existing trickplay images
  * Hacer clic en el botón `Refresh`

Esto debería hacer que, al salir y entrar en la librería, las películas de la colección se hayan desagregado:

![Películas en la colección][2]

Y que la aparezca correctamente imagen en la vista de Colecciones (en el siguiente ejemplo aún hay tres colecciones por arreglar):

![Colección con imagen][3]

### Historial de cambios

* **2025-08-17**: Documento inicial

[1]: /assets/img/blog/2025-08-17_image_1.png "Colección sin imagen"
[2]: /assets/img/blog/2025-08-17_image_2.png "Películas en la colección"
[3]: /assets/img/blog/2025-08-17_image_3.png "Colección con imagen"
