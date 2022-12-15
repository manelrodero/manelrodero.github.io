---
layout : post
blog-width: true
title: '¿Cómo incluir vídeos de YouTube en Markdown?'
date: '2022-12-15 22:49:09'
published: true
tags:
- Markdown
author:
  display_name: Manel Rodero
---

Como nunca me acuerdo cómo enlazar a un vídeo de YouTube en Markdown, voy a documentar diferentes maneras.

# Enlace

```markdown
[Comprehensive Markdown Crash Course](https://www.youtube.com/watch?v=FEa2diI2qgA)
```

Ejemplo:

[Comprehensive Markdown Crash Course](https://www.youtube.com/watch?v=FEa2diI2qgA)

# Enlace con título alternativo

```markdown
[Comprehensive Markdown Crash Course](https://www.youtube.com/watch?v=FEa2diI2qgA "Título alternativo")
```

Ejemplo:

[Comprehensive Markdown Crash Course](https://www.youtube.com/watch?v=FEa2diI2qgA "Título alternativo")

# Enlace con imágen

```markdown
[![Comprehensive Markdown Crash Course](https://img.youtube.com/vi/FEa2diI2qgA/mqdefault.jpg)](https://www.youtube.com/watch?v=FEa2diI2qgA)
```

Ejemplo:

[![Comprehensive Markdown Crash Course](https://img.youtube.com/vi/FEa2diI2qgA/mqdefault.jpg)](https://www.youtube.com/watch?v=FEa2diI2qgA)

# Enlace con imágen y texto alternativo

```markdown
[![Comprehensive Markdown Crash Course](https://img.youtube.com/vi/FEa2diI2qgA/mqdefault.jpg)](https://www.youtube.com/watch?v=FEa2diI2qgA "Título alternativo")
```

Ejemplo:

[![Comprehensive Markdown Crash Course](https://img.youtube.com/vi/FEa2diI2qgA/mqdefault.jpg)](https://www.youtube.com/watch?v=FEa2diI2qgA "Título alternativo")

# Iframe

* Abrir el vídeo de YouTube en el navegador
* Seleccionar las siguientes opciones: `Share` &rarr; `Embed`
* Marcar las opciones:
  * `Show player controls`
  * `Enable privacy-enhanced mode`
* Pulsar en `Copy` para copiar la *iframe*
* Pegarla en el código

```html
<iframe width="320" height="180" src="https://www.youtube-nocookie.com/embed/FEa2diI2qgA" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen="1"></iframe>
```

Ejemplo:

<iframe width="320" height="180" src="https://www.youtube-nocookie.com/embed/FEa2diI2qgA" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen="1"></iframe>

> Nota: en muchas implementaciones puede ser complicado insertar una _iframe_ directamente en el código. Por ejemplo, en Jekyll es necesario [añadir el valor `1` al atributo `allowfullscreen`](https://stackoverflow.com/a/12273878) para que sea compatible.

# API de terceros

Hay quien ha creado una API para obtener el _thumbnail_ del vídeo, añadirle un botón de reproducción y mostrar la imagen que se usará en el enlace:

```markdown
[![Comprehensive Markdown Crash Course](https://markdown-videos.deta.dev/youtube/FEa2diI2qgA)](https://www.youtube.com/watch?v=FEa2diI2qgA)
```

Ejemplo:

[![Comprehensive Markdown Crash Course](https://markdown-videos.deta.dev/youtube/FEa2diI2qgA)](https://www.youtube.com/watch?v=FEa2diI2qgA)

El proyecto [Video to markdown!](https://video-to-markdown.marcomontalbano.com/) de [Marco Montalbano](https://www.marcomontalbano.com/) hace algo similar.

```markdown
[![Comprehensive Markdown Crash Course](https://res.cloudinary.com/marcomontalbano/image/upload/v1671144387/video_to_markdown/images/youtube--FEa2diI2qgA-c05b58ac6eb4c4700831b2b3070cd403.jpg)](https://www.youtube.com/watch?v=FEa2diI2qgA "Comprehensive Markdown Crash Course")
```

# _includes

Otra opción para incrustar un vídeo es hacerlo mediante la creación de un fichero en el directorio `_includes` que contendrá el código `HTLM` que se **incluirá** en el código usando una sintaxis similar a ésta:

```markdown
{% raw %}
{% include youtubePlayer.html id="4EU7vvSvV-0" %}
{% endraw %}
```

Los siguientes artículos son diferentes implementaciones de esta idea:

* [Responsive YouTube Video with Jekyll](https://www.chunkhang.com/blog/responsive-youtube-video-with-jekyll)
* [How to Embed YouTube and Spotify on a GitHub Pages Blog](https://thisisa.blog/how-to-embed-media-github-pages)

# Referencias

* [Get YouTube Video Thumbnail Image](https://www.systoolsgroup.com/get-youtube-video-thumbnail-images/)
* [Writing Liquid Template in Markdown Code Blocks with Jekyll](https://ozzieliu.com/2016/04/26/writing-liquid-template-in-markdown-with-jekyll/)
