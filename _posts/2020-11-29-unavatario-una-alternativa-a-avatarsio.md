---
layout : post
blog-width: true
title: 'Unavatar.io, una alternativa a avatars.io'
date: '2020-11-29 21:30:11'
published: true
tags:
- Blog
author:
  display_name: Manel Rodero
---

En la sección [Blogroll](/community/blogroll) usaba **avatars.io** para obtener el _avatar_ del autor a partir su imagen de perfil en Twitter. Pero este servicio ha desaparecido, por lo que ha habido que buscar una alternativa.

[Unavatar.io](https://github.com/microlinkhq/unavatar), un desarrollo de código abierto muy sencillo de usar, puede obtener la imagen desde diferentes redes sociales:

* GitHub
* Facebook
* Gravatar
* Instagram
* Telegram
* YouTube
* SoundCloud
* Etc.

Por ejemplo, `https://unavatar.io/twitter/manelrodero` devuelve el _avatar_ de **@manelrodero** en Twitter.

Esta imagen se puede usar sin problema dentro de un elemento `<img>` como se muestra a continuación:

![@manelrodero](https://unavatar.io/twitter/manelrodero){:height="96px" width="96px"}
