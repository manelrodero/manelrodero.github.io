---
layout: post
blog-width: true
title: 'Links tipo UNIX en Windows'
date: '2005-10-17 17:35:25'
last-updated: '2025-03-01 21:28:16'
published: true
tags:
- Windows
author:
  display_name: Manel Rodero
---

Hoy me he encontrado con el siguiente problema: los drivers y las aplicaciones de la impresora **HP Officejet 7310** no se dejan instalar en `D:\SOFT\HP`. Después de un rato copiando archivos, se interrumpe el proceso de instalación y aparece un error del módulo **AiO_Scan** indicando que "_no puede crear el directorio D:\SOFT\HP_".

Lo curioso es que el programa de instalación sí que ha creado este directorio y ha copiado archivos en él desde el inicio de la instalación (cuando se le indicó que ésta era la ruta que queríamos para instalar el software).

Lo malo de este problema, es que no quiero instalar el programa en la unidad `C:\` para no llenarla (la instalación completa ocupa unos 800MB) por lo que he decidido buscar alguna manera de engañar al programa de instalación.

La idea es indicarle al programa que instale en un directorio de la unidad `C:\` pero que éste realmente sea un enlace a un directorio de la unidad `D:\` para que "se gaste espacio" de esta última unidad. Es decir, quiero conseguir algo así:

```plaintext
C:\D\SOFT –> D:\SOFT
```

Buscando, buscando, he encontrado el [siguiente artículo de Shell-Shocked.org](https://web.archive.org/web/20050909025458/http://shell-shocked.org/article.php?id=284){:target="_blank"} relacionado con los enláces simbólicos bajo Windows, cómo funcionan, qué herramientas hay para crearlos, etc. que me ha parecido bastante interesante.

En el artículo anterior se menciona la utilidad `junction.exe` de [SysInternals](https://learn.microsoft.com/en-us/sysinternals/){:target="_blank"} mediante la cual se pueden crear los enlaces entre unidades (en Windows XP existe una utilidad llamada `fsutil.exe` para hacer este tipo de enlaces pero únicamente funciona entre ficheros y directorios de la misma unidad).

¡La instalación ha funcionado perfectamente!

### Historial de cambios

* **2005-10-17**: Documento inicial
* **2025-02-28**: Corrección de enlaces (WebArchive.org)
