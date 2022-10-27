---
layout : post
blog-width: true
title: 'Instalación personalizada de Adobe Acrobat Reader'
date: '2022-06-21 15:16:55'
published: true
tags:
- Software
author:
  display_name: Manel Rodero
---

> **27 de Octubre de 2022**: revisión de las versiones y la validez de las instrucciones.

[Adobe Acrobat Reader](https://www.adobe.com/acrobat/pdf-reader.html) es un programa gratuito para ver, firmar, colaborar y anotar archivos PDF.

No hay que confundirlo con [Adobe Acrobat](https://www.adobe.com/acrobat.html), el programa comercial que permite crear y editar archivos PDF así como convertirlos a formatos de Microsoft Office.

![Acrobat Reader][1]
![Acrobat][2]

En este artículo se describe cómo realizar una instalación **desatendida** de Adobe Acrobat Reader DC configurándolo de manera **personalizada** mediante el uso de [_Adobe Customization Wizard_](https://www.adobe.com/devnet-docs/acrobatetk/tools/Wizard/index.html).

# ¿Cuál es la versión actual?

Para conocer cuál es la versión actual de **Acrobat y Acrobat Reader** se puede acceder a la página de las [_Release Notes_](https://www.adobe.com/devnet-docs/acrobatetk/tools/ReleaseNotesDC/index.html) de este programa.

Se instalará la última versión correspondiente al [**Continuous Track**](https://www.adobe.com/devnet-docs/acrobatetk/tools/AdminGuide/whatsnewdc.html) que, a día de hoy, es la `22.003.20263` del 22 de Octubre de 2022.

Se ha elegido el _Continuous Track_ y no el _Classic Track_ para poder disponer de nuevas características, mejoras de seguridad y plataforma, y correcciones de errores como parte de actualizaciones silenciosas más frecuentes.

La [numeración de las versiones](https://www.adobe.com/devnet-docs/acrobatetk/tools/AdminGuide/whatsnewdc.html#product-versioning) de Adobe Acrobat Reader tiene el siguiente formato:

* Año de liberación: `22`
* _Build_ interna: `003`
* Identificador del _track_:
  * `20` (_Continuous_)
  * `30` (_Classic_)
* _Build_ interna: `263`
* Lista de cambios: `xxxxx` (oculto, únicamente se ve en el _About_)

# Instalador Acrobat Reader

Para descargar el instalador se accede a la página de [distribución corporativa de Adobe Acrobat Reader](https://get.adobe.com/es/reader/enterprise/) y se selecciona el sistema operativo, el idioma y la versión:

![Download][3]

En este caso se han seleccionado las siguientes opciones:

* Sistema operativo: `Windows 10` (también se podría escoger Windows 11)
* Idioma: `Spanish`
* Versión: `Reader DC 2022.003.20263 Spanish for Windows`

A continuación se puede pulsar sobre el botón azul `Descargar Acrobat Reader` para descargar el instalador:

![Download][4]

> **Nota**: Podemos saltarnos el paso 2 de la página web ya que únicamente nos interesa obtener el fichero con el instalador [`AcroRdrDC2200320263_es_ES.exe`](https://ardownload2.adobe.com/pub/adobe/reader/win/AcrobatDC/2200320263/AcroRdrDC2200320263_es_ES.exe).

> **Nota**: Si se selecciona `All languages (MUI)`, entonces es posible elegir la versión `Reader DC 2022.003.20263 MUI for Windows-64bit` y descargar un instalador de 64-bits [`AcroRdrDCx642200320263_MUI.exe`](https://ardownload2.adobe.com/pub/adobe/acrobat/win/AcrobatDC/2200320263/AcroRdrDCx642200320263_MUI.exe).

![Adobe Reader MUI][6]

# Carpeta de instalación

La creación de la carpeta desde la cual se realizará la instalación del programa es muy sencillo, únicamente es necesario crear el directorio y extraer el contenido del instalador:

```Batch
mkdir G:\ADOBE\Reader
AcroRdrDC2200320263_es_ES.exe -nos_o"G:\ADOBE\Reader" -nos_ne
```

Esta carpeta contendrá los ficheros correspondientes al **instalador** (`setup.exe`), los parámetros de instalación (`abcpy.ini` y `setup.ini`) y la actualización correspondiente a la versión descargada (en este caso `AcroRdrDCUpd2200320263.msp`):

```
 Directory of G:\ADOBE\Reader

10/27/2022  07:25 PM    <DIR>          .
10/27/2022  07:25 PM    <DIR>          ..
16/10/2022  22:43               605 abcpy.ini
16/10/2022  22:43       413.716.480 AcroRdrDCUpd2200320263.msp
17/03/2015  10:41         2.806.272 AcroRead.msi
17/03/2015  10:35       179.823.429 Data1.cab
16/10/2022  22:43           526.288 setup.exe
16/10/2022  22:43                92 setup.ini
               6 File(s)    596.873.166 bytes
```

# Acrobat Customization Wizard

Si se ejecutase el `setup.exe` del directorio anterior, aparecería el asistente de instalación típico de un programa. En este asistente habría que indicar los componentes a instalar, dónde se hará la instalación, etc.

Ahora bien, en este caso se **personalizará** la instalación para hacerla **desatendida** y configurar algunas características del programa (por ejemplo, las actualizaciones o la seguridad).

Para ello es necesario descargar e instalar la última versión disponible de [Acrobat Customization Wizard](https://www.adobe.com/devnet-docs/acrobatetk/tools/Wizard/index.html). A día de hoy es [`CustWiz2100720091_en_US_DC.exe`](https://ardownload2.adobe.com/pub/adobe/acrobat/win/AcrobatDC/misc/CustWiz2100720091_en_US_DC.exe).

# Personalización

Para personalizar la instalación, se ejecuta el asistente (`"C:\Program Files (x86)\Adobe\Acrobat Customization Wizard DC\CustWiz.exe"`) y se procede a modificar el contenido de la carpeta de instalación.

* File > Open Package...
* Seleccionar el paquete `"G:\ADOBE\Reader\AcroRead.msi"`

Una vez abierto el paquete, el asistente mostrará las diferentes secciones de configuración en el menú lateral izquierdo. Al pulsar sobre cada una de ellas, aparecerán los posibles valores a configurar en la parte derecha del asistente. En la parte inferior se muestra la ayuda correspondiente a cada sección.

![Acrobat Customization Wizard][5]

A continuación se detallan los valores que hemos utilizado desde hace años en los ordenadores del PDI/PAS de la [Facultat d'Informàtica de Barcelona](https://www.fib.upc.edu/).

## Personalization Options

- Installation Path: Dejar `<Default Installation Path>`
- EULA Option: Marcar `Suppress display of End User License Agreement (EULA)`

## Installation Options

- Default viewer for PDF files: Seleccionar `Make Reader the default PDF viewer`
- Dejar marcado `Remove all versions of Reader`
- Dejar marcado `Enable Optimization`
- Run Installation: Seleccionar `Silently (no interface)`
- If reboot required at the end of installation: Seleccionar `Supress reboot`

> Si se está personalizando la versión **MUI**, se podría escoger el lenguaje en que se desea realizar la instalación (por defecto es el `OS Language`).


## Files and Folders

## Registry

## Shortcuts

- Start Menu > Programs > Dejar `Acrobat Reader DC`
- Desktop > `Acrobat Reader DC` > Seleccionar `Remove`

## Server Locations

## Security

- Protected View: Seleccionar `All files`
- Enhanced Security Settings:
  - Standalone = Cambiar `Default` a `Enable`
  - Browser = Cambiar `Default` a `Enable`

## Digital Signature

- Verification
  - Dejar marcado `Verify signatures when the document is opened`
- Creation
  - Marcar `Show location and contact information when signing`
  - Prevent signing until document warnings are reviewed: Cambiar `Never` a `When Certifying Only`
  - Marcar `Show reasons when signing`

## WebMail Profiles

## Online Services and Features

- Dejar desmarcado `Disable product updates` para tener [actualizaciones automáticas](https://helpx.adobe.com/acrobat/kb/automatic-updates---acrobat-reader.html)
- Load trusted root certificates from Adobe: Cambiar `Enable & Ask before installing` a `Enabled & install silently`
- Marcar `Disable upsell`
- Marcar `Disable all Adobe services` para deshabilitar los siguientes servicios:
	- Adobe Acrobat Document Cloud services
	- Adobe Sign
	- Send for Review
	- Preference synchronization across devices
- Marcar `Disable third party connectors such as Dropbox, Google Drive, etc.`
- Marcar `Disable SharePoint connector`

## Comments and Forms

## File Attachments

- Marcar `Prevent document from opening other files and launching other applications`

## Launch Other Applications

## Other Settings

- Marcar `Enable Save Ink/Toner`
- Marcar `Prevent the end user from enabling PDF preview in Windows Explorer`

## Direct Editor

# Grabación del paquete

Una vez se han revisado las diferentes opciones de configuración, se puede **grabar el paquete** usando la opción `File > Save Package` y salir del programa.

La grabación del paquete modificará el contenido de la carpeta de instalación agregando un fichero de transformación `AcroRead.mst` con la configuración escogida en el asistente:

```
 Directory of G:\ADOBE\Reader

10/27/2022  07:25 PM    <DIR>          .
10/27/2022  07:25 PM    <DIR>          ..
10/16/2022  10:43 PM               605 abcpy.ini
10/16/2022  10:43 PM       413,716,480 AcroRdrDCUpd2200320263.msp
03/17/2015  10:41 AM         2,806,272 AcroRead.msi
10/27/2022  06:47 PM            45,056 AcroRead.mst
03/17/2015  10:35 AM       179,823,429 Data1.cab
10/16/2022  10:43 PM           526,288 setup.exe
10/27/2022  06:47 PM               146 setup.ini
               7 File(s)    596,918,276 bytes
```

Además, se modificará el fichero `setup.ini` para indicar que la instalación use el fichero de transformación:

```
[Startup]
RequireMSI=3.0
CmdLine=/sall /rs

[Product]
PATCH=AcroRdrDCUpdd2200320263.msp
msi=AcroRead.msi
CmdLine=TRANSFORMS="AcroRead.mst"
```

# Instalación (desatendida)

Para instalar Adobe Acrobat Reader DC de forma desatendida con las opciones de configuración elegidas únicamente hay que ejecutar el `setup.exe` de la carpeta de instalación sin ningún otro parámetro.

# Referencias

* [Bootstrapper](https://www.adobe.com/devnet-docs/acrobatetk/tools/DesktopDeployment/bootstrapper.html), opciones del fichero `setup.ini` y la línea de comandos
* [Properties](https://www.adobe.com/devnet-docs/acrobatetk/tools/AdminGuide/properties.html), las opciones `USERNAME` y `COMPANYNAME` se deben especificar en la línea de comandos
* [PDF Viewer](https://www.adobe.com/devnet-docs/acrobatetk/tools/AdminGuide/pdfviewer.html), para forzar un visor de PDF concreto, se puede crear un fichero XML e inyectarlo en el sistema operativo mediante DISM o GPO
* [SCCM-SCUP](https://www.adobe.com/devnet-docs/acrobatetk/tools/DesktopDeployment/sccm.html), uso de System Center Updates Publisher para instalar y actualizar los productos de Adobe

[1]: /assets/img/blog/2022-06-21_image_1.png "Acrobat Reader"
[2]: /assets/img/blog/2022-06-21_image_2.png "Acrobat"
[3]: /assets/img/blog/2022-06-21_image_3.png "Download"
[4]: /assets/img/blog/2022-06-21_image_4.png "Download"
[5]: /assets/img/blog/2022-06-21_image_5.png "Acrobat Customization Wizard"
[6]: /assets/img/blog/2022-06-21_image_6.png "Adobe Reader MUI"
