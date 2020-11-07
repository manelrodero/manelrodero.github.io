---
layout: post
title: Actualización a Firefox 1.5.0.1
date: 2006-02-04 21:18:24.000000000 +01:00
published: true
tags:
- Software
author:
  display_name: Manel Rodero
---

Hace tiempo que quería actualizar mi Firefox (un 1.0.6) a la nueva familia 1.5.x pero había un par de cosas que me frenaban:

* **Rendimiento**: al parecer la versión 1.5 había sido un desastre en cuanto a rendimiento se refiere y los que se habían atrevido a instalarlo estaban muy descontentos con su funcionamiento. La búsqueda "[firefox 1.5 memory][2]" en Google proporciona gran cantidad de información sobre estos problemas.
* **Extensiones**: no todas las extensiones que uso habitualmente eran compatibles con la nueva versión de Firefox. Había que esperar a que los autores de las mismas tuviesen tiempo y ganas de actualizarlas.

![Mozilla Firefox 1.5][1]

Ayer, cuando finalmente me puse a ello, comprobé que ya había aparecido la versión 1.5.0.1 de Firefox (se supone que para solucionar el tema del rendimiento comentado anteriormente) y que la gran mayoría de extensiones que uso habitualmente también se habían actualizado para ser compatibles con la nueva família.

Las extensiones compatibles son:

* **[Adblock][3] v0.5.3.042**: Sirve para filtrar contenidos no deseados (banners, iframes, flash, contenido para adultos, etc.) en tiempo real mientras se navega por Internet.
* **[Adblock Filterset.G Updater][4] 0.3.0.1**: Actualizador automático de los filtros Filterset.G: un conjunto de filtros diseñados para eliminar la mayoría de publicidad no deseada en Internet.
* **[BugMeNot][5] 1.3**: Busca automáticamente un usuario y contraseña para las páginas web que necesitan un registro previo antes de entrar o descargar algo de ellas.
* **[DownThemAll!][6] 0.9.8.7**: Sirve para añadir funcionalides de descarga avanzadas a Firefox, como por ejemplo, poder descargar de un solo click todas las imágenes de una página.
* **[Googlebar][7] 0.9.15.08**: Barra de herramientas que añade funcionalides de Google (búsquedas, traducción, diccionarios, etc.) a Firefox.
* **[IE View][8] 1.2.7**: Permite abrir en Internet Explorer la página actual o un determinado enlace que únicamente se visualizan correctamente con este último navegador.
* **[PDF Download][9] 0.6**: Sirve para evitar que Firefox ejecute Adobe u otro plugin similar para intentar mostrar el fichero PDF dentro del navegador.
* **[Flat Bookmark Editing][10] 0.7.5**: Proporciona acceso a la edición de las propiedades de un enlace desde el propio gestor de enlaces de Firefox (sin tener que usar el botón derecho en cada uno de los enlaces).

Las extensiones no compatibles (a fecha de hoy) son:

* **[Bookmark Tags][11] 0.2.0**: Permite organizar los enlaces mediante tags (categorías) en lugar del método tradicional por carpetas.

El procedimiento que he seguido para actualizar Firefox es el habitual, es decir, **he comenzado desde cero para evitar problemas posteriores** con las extensiones o parámetros incompatibles entre versiones.

La ventaja de este método es evidente: se dispondrá de una instalación limpia del navegador por lo que los problemas que pudieran surgir serán debidos a lo que se haga desde ahora y no a algo heredado de la antigua instalación.

La desventaja: obliga a reconfigurar desde cero todo el perfil de usuario y las extensiones (aunque esto es un problema menor porque normalmente anoto todos estos detalles y algunos aspectos, como los _bookmarks_, se pueden exportar antes de comenzar el proceso).

El procedimiento que he seguido es el siguiente:

* Entrar con la cuenta "Administrator" (la cuenta "Manel" no tiene permisos)
* Desinstalar Mozilla Firefox 1.0.6 desde el panel de control de Windows
* Eliminar restos de la instalación borrando `G:\Program Files\Mozilla Firefox`
* Renombrar el directorio que contiene el antiguo perfil `F:\Manel\Mozilla\Firefox` a `F:\Manel\Mozilla\Firefox.old`
* Ejecutar el instalador "`Firefox Setup 1.5.0.1.exe`"
* Tipo de instalación: Custom
* Directorio de instalación: `G:\Program Files\Mozilla Firefox`
* Developer Tools: Instalar
* Quality Feedback Agent: No instalar
* Desktop: No crear icono
* Quick Launch Bar: No crear icono
* Start Menu: Crear icono
* Arreglar menú de inicio (grupo "Mozilla Firefox" a "Software XARXA")
* Arreglar permisos del menú de inicio (heredar en todas las carpetas)
* Ejecutar `firefox.exe -p` y comprobar que el perfil por defecto sigue siendo `F:\Manel\Mozilla\Firefox`
* Ejecutar `firefox.exe`, modificar la configuración y salir del programa
* Hacer un backup del perfil que se acaba de crear para poder usarlo si hay problemas en el futuro con alguna extensión (a veces la instalación de una extensión no muy bien programada puede dejar el perfil inservible y, por tanto, sin posibilidad de arrancar Firefox o dejándolo un tanto inestable)

La configuración que suelo usar es la siguiente:

* General – Home Page = Blank
* General – Default Browser = Shouldn't check (Firefox ya es mi navegador por defecto)
* Privacy – History = 15 días
* Privacy – Saved Forms = Don't save
* Privacy – Passwords = Don't remember
* Privacy – Download history = Remove manually
* Privacy – Clear Private Data = Todos los elementos
* Content – Java Script = Don't disable/replace context menus
* Tools – Open links = In a new window
* Downloads – Folder = Ask
* Downloads – Download Manager = Close when complete
* Advanced – General = Don't resize images
* Advanced – Update = Ask me

Antes de comenzar la instalación de las extensiones he realizado lo siguiente:

Finalmente, he procedido a instalar las extensiones comentadas anteriormente **una a una** y reiniciando Firefox cada vez para comprobar el correcto funcionamiento de las mismas y del navegador. Además, he recuperado el antiguo fichero de enlaces `bookmark.htm` y los filtros del Adblock que había guardado antes de comenzar todo el proceso.

De momento no he notado ninguna pérdida de rendimiento ni problemas con la memoria tal como parece que pasaba con la versión 1.5. De todas maneras, aún me falta realizar algunos ajustes en la configuración "oculta" de Firefox (a la que se accede mediante `about:config`) para acelerar la descarga de páginas o el arranque del programa la primera vez que se ejecuta. Pero esto será otro día.

[1]: /assets/img/blog/2006-02-04_image_1.png "Mozilla Firefox 1.5"
[2]: http://www.google.com/search?sourceid=mozclient&ie=utf-8&oe=utf-8&q=firefox+1%2E5+memory
[3]: http://adblock.mozdev.org/
[4]: http://www.pierceive.com/
[5]: http://roachfiend.com/archives/2005/02/07/bugmenot/
[6]: http://downthemall.mozdev.org/
[7]: http://googlebar.mozdev.org/
[8]: http://ieview.mozdev.org/
[9]: http://www.rabotat.org/firefox/
[10]: http://bluweb.com/us/chouser/proj/mozhack/
[11]: http://www.arches.uga.edu/~dripfeed/bookmarktags/
