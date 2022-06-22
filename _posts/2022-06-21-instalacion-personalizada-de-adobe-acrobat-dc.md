---
layout : post
blog-width: true
title: 'Instalación personalizada de Adobe Acrobat DC'
date: '2022-06-22 17:26:22'
published: true
tags:
- Software
author:
  display_name: Manel Rodero
---

[Adobe Acrobat](https://www.adobe.com/acrobat.html) es un programa comercial que permite crear y editar archivos PDF así como convertirlos a formatos de Microsoft Office.

No hay que confundirlo con [Adobe Acrobat Reader](https://www.adobe.com/acrobat/pdf-reader.html), el programa gratuito que permite ver, firmar, colaborar y anotar archivos PDF.

Actualmente existen [dos versiones](https://www.adobe.com/acrobat/pricing/compare-versions.html) de Adobe Acrobat, la **Standard** y la **Pro**. Estas versiones se pueden comprar en versión **Perpetual** (Acrobat 2017 y Acrobat 2020) o mediante una subscripción **Continuous** (Acrobat DC):

![Pro vs Standard][1]

En este artículo se describe cómo realizar una instalación **desatendida** de Adobe Acrobat DC configurándolo de manera **personalizada** mediante el uso de [_Adobe Customization Wizard_](https://www.adobe.com/devnet-docs/acrobatetk/tools/Wizard/index.html).

# ¿Cuál es la versión actual?

Para conocer cuál es la versión actual de **Acrobat y Acrobat Reader DC** se puede acceder a la página de las [_Release Notes_](https://www.adobe.com/devnet-docs/acrobatetk/tools/ReleaseNotesDC/index.html) de este programa.

Se instalará la última versión correspondiente al [**Continuous Track**](https://www.adobe.com/devnet-docs/acrobatetk/tools/AdminGuide/whatsnewdc.html) que, a día de hoy, es la `22.001.20142` del 14 de Junio de 2022.

Se ha elegido el _Continuous Track_ y no el _Classic Track_ para poder disponer de nuevas características, mejoras de seguridad y plataforma, y correcciones de errores como parte de actualizaciones silenciosas más frecuentes.

![Continuous vs Classic][3]

# Instalador Acrobat DC

El instalador se puede descargar desde la página de [licencia Enterprise Term o VIP](https://helpx.adobe.com/es/acrobat/kb/acrobat-dc-downloads.html) seleccionando el botón correspondiente a la versión de sistema operativo que se requiera.

En este caso se pulsa sobre el botón azul `Descargar` correspondiente al sistema operativo **Windows** y se descarga el fichero `Acrobat_DC_Web_WWMUI.zip` que habrá que descomprimir para obtener el instalador.

# Carpeta de instalación

La creación de la carpeta desde la cual se realizará la instalación del programa es muy sencillo, únicamente es necesario crear el directorio y extraer el contenido del instalador:

```Batch
unzip.exe -d G:\TEMP Acrobat_DC_Web_WWMUI.zip
move "G:\TEMP\Adobe Acrobat" G:\ADOBE\Acrobat
ren "G:\ADOBE\Acrobat\Adobe Acrobat" DC
```

Esta carpeta contendrá los ficheros correspondientes al **instalador** (`setup.exe`), los parámetros de instalación (`abcpy.ini` y `setup.ini`) y la actualización correspondiente a la versión descargada (en este caso `AcrobatDCUpd2200120142.msp`):

```
 Directorio de G:\ADOBE\Acrobat\DC

22/06/2022  09:47    <DIR>          .
22/06/2022  09:47    <DIR>          ..
04/06/2022  20:28               647 ABCPY.INI
04/06/2022  20:28       507.592.704 AcrobatDCUpd2200120142.msp
04/06/2022  20:28        12.911.616 AcroPro.msi
04/06/2022  20:28       567.178.357 Data1.cab
04/06/2022  20:28           524.056 Setup.exe
04/06/2022  20:29             1.361 setup.ini
22/06/2022  09:47    <DIR>          Transforms
22/06/2022  09:47    <DIR>          VCRT_x64
04/06/2022  20:28         2.585.872 WindowsInstaller-KB893803-v2-x86.exe
               7 archivos  1.090.794.613 bytes
```

# Acrobat Customization Wizard

Si se ejecutase el `setup.exe` del directorio anterior, aparecería el asistente de instalación típico de un programa. En este asistente habría que indicar los componentes a instalar, dónde se hará la instalación, etc.

Ahora bien, en este caso se **personalizará** la instalación para hacerla **desatendida** y configurar algunas características del programa (por ejemplo, las actualizaciones o la seguridad).

Para ello es necesario descargar e instalar la última versión disponible de [Acrobat Customization Wizard](https://www.adobe.com/devnet-docs/acrobatetk/tools/Wizard/index.html). A día de hoy es [`CustWiz2100720091_en_US_DC.exe`](https://ardownload2.adobe.com/pub/adobe/acrobat/win/AcrobatDC/misc/CustWiz2100720091_en_US_DC.exe).

# Personalización

Para personalizar la instalación, se ejecuta el asistente (`"C:\Program Files (x86)\Adobe\Acrobat Customization Wizard DC\CustWiz.exe"`) y se procede a modificar el contenido de la carpeta de instalación.

* File > Open Package...
* Seleccionar el paquete `"G:\ADOBE\Acrobat\DC\AcroPro.msi"`

Una vez abierto el paquete, el asistente mostrará las diferentes secciones de configuración en el menú lateral izquierdo. Al pulsar sobre cada una de ellas, aparecerán los posibles valores a configurar en la parte derecha del asistente. En la parte inferior se muestra la ayuda correspondiente a cada sección.

![Acrobat Customization Wizard DC][2]

A continuación se detallan los valores que hemos utilizado desde hace años en los ordenadores del PDI/PAS de la [Facultat d'Informàtica de Barcelona](https://www.fib.upc.edu/).

## Personalization Options

- Serial Number: Introducir `xxxx-xxxx-xxxx-xxxx-xxxx-xxxx`
- Pulsar el botón `Grant Offline Exception & Disable Registration`
- Installation Path: Dejar `<Default Installation Path>`
- EULA Option: Marcar `Suppress display of End User License Agreement (EULA)`

> **Nota**: El uso de un número de serie modifica la variable `ISX_SERIALNUMBER`. Al hacer el _grant_ se modifica la variable `PRESERIALIZATIONFILEPATH` que permite que el producto funcione sin activar si no hay conexión. Si hay conexión, se activa el producto en Adobe y se elimina la opción de registro `Help > Register Product`.

## Installation Options

- Default viewer for PDF files: Dejar `Installer will decide which product will be the default`
- Dejar `Remove previous version of Acrobat`
- No marcar `Remove all versions of Reader`
- Run Installation: Seleccionar `Silently (no interface)`
- If reboot required at the end of installation: Seleccionar `Supress reboot`
- Application Language: `Spanish (Traditional Sort)`

## Features

- Create Adobe PDF
  - Acrobat PDFMaker: `The Feature will not be available`

## Files and Folders

## Registry

## Shortcuts

- Start Menu > Programs > Acrobat Acrobat DC (dejar)
- Start Menu > Programs > Acrobat Acrobat Distiller DC (dejar)
- Desktop > Acrobat Acrobat DC &rarr; `Remove`

## Server Locations

## PDF Creation

- Desmarcar `View Adobe PDF results`
- Marcar `Ask to replace existing PDF file`
- Desmarcar `Add document information`
- Desmarcar `Rely on system fonts; do not use document fonts`

## Security

- Protected View: Seleccionar `All files`
- Enhanced Security Settings:
  - Standalone = Cambiar `Default` a `Enable`
  - Browser = Cambiar `Default` a `Enable`

## Digital Signature

- Marcar `Show location and contact information when signing`
- Prevent signing until document warnings are reviewed: Cambiar `Never` a `When Certifying Only`
- Marcar `Show reasons when signing`

## Rights Management Servers

## WebMail Profiles

## Online Services and Features

- Dejar desmarcado `Disable product updates` para tener [actualizaciones automáticas](https://helpx.adobe.com/acrobat/kb/automatic-updates---acrobat-reader.html)
- Load trusted root certificates from Adobe: Cambiar `Enable & Ask before installing` a `Enabled & install silently`
- Marcar `Disable upsell`
- Marcar `Disable all Adobe services` deshabilitará los siguientes servicios:
	- Adobe Acrobat Document Cloud services
	- Adobe Sign
	- Send for Review
	- Preference synchronization across devices
- Marcar `Disable third party connectors such as Dropbox, Google Drive, etc.`
- Marcar `Disable SharePoint connector`

## Comments and Forms

## File Attachments

- Marcar `Prevent document from opening other files and launching other applications`

## Header/Footer, Watermark & Background

## Protection

- Marcar `Remove Hidden Information when closing document`
- Marcar `Remove Hidden Information when sending document by email`

## Launch Other Applications

## Tools

## Other Settings

- Marcar `Enable Save Ink/Toner`
- Marcar `Prevent the end user from enabling PDF preview in Windows Explorer`

## Direct Editor

# Grabación del paquete

Una vez se han revisado las diferentes opciones de configuración, se puede **grabar el paquete** usando la opción `File > Save Package` y salir del programa.

La grabación del paquete modificará el contenido de la carpeta de instalación agregando un fichero de transformación `AcroPro.mst` con la configuración escogida en el asistente:

```
 Directorio de G:\ADOBE\Acrobat\DC

22/06/2022  10:17    <DIR>          .
22/06/2022  10:17    <DIR>          ..
04/06/2022  20:28               647 ABCPY.INI
04/06/2022  20:28       507.592.704 AcrobatDCUpd2200120142.msp
04/06/2022  20:28        12.911.616 AcroPro.msi
22/06/2022  10:17           270.336 AcroPro.mst
22/06/2022  10:17        13.119.488 AcroPro.ref
04/06/2022  20:28       567.178.357 Data1.cab
04/06/2022  20:28           524.056 Setup.exe
22/06/2022  10:17             1.415 setup.ini
22/06/2022  09:47    <DIR>          Transforms
22/06/2022  09:47    <DIR>          VCRT_x64
04/06/2022  20:28         2.585.872 WindowsInstaller-KB893803-v2-x86.exe
               9 archivos  1.104.184.491 bytes
```

Además, se modificará el fichero `setup.ini` para indicar que la instalación use el fichero de transformación:

```
[Startup]
RequireOS=Windows 7
RequireOS64=Windows 10
RequireMSI=3.1
RequireIE=7.0.0000.0
Require64BitVCRT=1
CmdLine= /sall /rs

[Product]
PATCH=AcrobatDCUpd2200120142.msp
msi=AcroPro.msi
vcrtMsi=vc_runtimeMinimum_x64.msi
vcrtDir=VCRT_x64
vcrtCommandLine=VSEXTUI=1
Languages=2052;1028;1029;1030;1043;1033;1035;1036;1031;1038;1040;1041;1042;1044;1045;1046;1049;1051;1060;1034;1053;1055;1058;1025;1037;6156
2052=Chinese Simplified
1028=Chinese Traditional
1029=Czech
1030=Danish
1043=Dutch (Netherlands)
1033=English (United States)
1035=Finnish
1036=French (France)
1031=German (Germany)
1038=Hungarian
1040=Italian (Italy)
1041=Japanese
1042=Korean
1044=Norwegian (Bokmal)
1045=Polish
1046=Portuguese (Brazil)
1049=Russian
1051=Slovak
1060=Slovenian
1034=Spanish (Traditional Sort)
1053=Swedish
1055=Turkish
1058=Ukrainian
1025=English with Arabic support
1037=English with Hebrew support
6156=French (Morocco)
CmdLine=TRANSFORMS="AcroPro.mst"

[PatchProduct1]
ProductType=Acrobat
PatchVersion=11.0.12
Path=AcrobatUpd11012.msp
IgnoreFailure=1

[PatchProduct2]
ProductType=Acrobat
PatchVersion=10.1.16
Path=AcrobatUpd10116.msp
IgnoreFailure=1

[PatchProduct3]
ProductType=Acrobat
PatchVersion=15.006.30352
Path=Acrobat2015Upd1500630352.msp

[Windows 7]
PlatformID=2
MajorVersion=6
MinorVersion=1

[Windows 10]
PlatformID=2
MajorVersion=10
```

# Instalación (desatendida)

Para instalar Adobe Acrobat DC de forma desatendida con las opciones de configuración elegidas únicamente hay que ejecutar el `setup.exe` de la carpeta de instalación.

# Referencias

* [Bootstrapper](https://www.adobe.com/devnet-docs/acrobatetk/tools/DesktopDeployment/bootstrapper.html), opciones del fichero `setup.ini` y la línea de comandos
* [Properties](https://www.adobe.com/devnet-docs/acrobatetk/tools/AdminGuide/properties.html), las opciones `USERNAME` y `COMPANYNAME` se deben especificar en la línea de comandos
* [PDF Viewer](https://www.adobe.com/devnet-docs/acrobatetk/tools/AdminGuide/pdfviewer.html), para forzar un visor de PDF concreto, se puede crear un fichero XML e inyectarlo en el sistema operativo mediante DISM o GPO
* [SCCM-SCUP](https://www.adobe.com/devnet-docs/acrobatetk/tools/DesktopDeployment/sccm.html), uso de System Center Updates Publisher para instalar y actualizar los productos de Adobe
* [Acrobat Downloads](https://helpx.adobe.com/es/download-install/kb/acrobat-downloads.html), descarga de Adobe Acrobat DC, 2020 y 2017

[1]: /assets/img/blog/2022-06-22_image_1.png "Pro vs Standard"
[2]: /assets/img/blog/2022-06-22_image_2.png "Acrobat Customization Wizard DC"
[3]: /assets/img/blog/2022-06-22_image_3.png "Continuous vs Classic"
