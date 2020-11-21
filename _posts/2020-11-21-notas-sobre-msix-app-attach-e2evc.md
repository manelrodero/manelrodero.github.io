---
layout : post
blog-width: true
title: 'Notas sobre MSIX app attach (E2EVC)'
date: '2020-11-21 13:40:34'
published: true
tags:
- Windows
- MSIX
- PowerShell
author:
  display_name: Manel Rodero
share-img: "/assets/img/blog/2020-11-21_cover.png"
---

Durante los días 20 y 21 de noviembre de 2020 se está celebrando el evento **E2EVC Hybrid EUC Conference** de [E2EVC](https://twitter.com/E2EVC) en el que [Christiaan Brinkhoff](https://twitter.com/Brinkhoff_C) ha dado una charla titulada "**Microsoft MSIX app attach – Get To Know everything about It**".

El evento se está realizando mediante la plataforma GoToWebinar y en los próximos días se espera que se publiquen las grabaciones de las sesiones.

Este artículo es un resumen de las ideas y conceptos explicados durante el la presentación en el orden en el que han ido apareciendo. No pretende ser algo estructurado y con una buena redacción, sino unas notas rápidas con todo aquello que será interesante investigar.

## ¿Qué es MSIX?

Antes de empezar con la asociación de aplicaciones MSIX ([_MSIX app attach_](https://docs.microsoft.com/en-us/azure/virtual-desktop/what-is-app-attach)), la tecnología que está evolucionando como una forma de entregar las aplicaciones en [Windows Virtual Desktop](https://docs.microsoft.com/en-us/azure/virtual-desktop/overview), es necesario conocer [**qué es MSIX**](https://docs.microsoft.com/en-us/windows/msix/overview).

MSIX es una tecnología de **empaquetado de aplicaciones** que va mucho más allá de un simple `setup.exe` o del uso de un paquete `*.msi`. Utiliza algunos conceptos heredados de la [virtualización de aplicaciones (App-V)](https://docs.microsoft.com/en-us/windows/application-management/app-v/appv-getting-started) y, mediante el uso de PowerShell, permite empaquetar una aplicación en un contenedor que permite desplegarlas de forma fácil tanto en entornos _on prem_ como en la nube.

Las características clave de este formato son:

* **Fiabilidad**: asegura la instalación en un 99.96% y una desinstalación completa sin dejar rastro
* **Optimización del ancho de banda**: usa únicamente bloques de 64k y es un formato preparada para el uso desde la nube
* **Optimización del espacio en disco**: evita la duplicación de ficheros entre aplicaciones manteniendo la independencia de cada una de ellas

La herramienta [**MSIX Packaging Tool**](https://docs.microsoft.com/en-us/windows/msix/packaging-tool/tool-overview) permite convertir cualquier aplicación Win32, de forma interactiva (a través de la UI) o desde la línea de comandos, que tengamos en los siguientes formatos:

* `*.exe`
* `*.msi`
* App-V v5.x
* Click-Once

Para usar esta herramienta se necesita Windows 10 1809 o superior y tener permisos de administrador en la máquina. Aunque no es necesario, se recomienda disponer de una cuenta Microsoft (persoonal o work) para acceder a la Microsoft Store y descargarla.

## Tipos de imágenes

Tradicionalmente, a la hora de desplegar una imagen de Windows a diferentes tipos de usuarios, suele usarse alguna de las siguientes aproximaciones:

* **Legacy**: diferentes imágenes según el uso de las mismas (la imagen para contabilidad, la imagen para desarrolladores, etc.)
* **Standard**: una única imágen monolítica con todas las aplicaciones y, en algunos casos, usando _app masking_ para ocultar las aplicaciones según la persona que la usa
* **Modern**: una imagen sin aplicaciones donde éstas se integran mediante soluciones de _streaming_ o ahora mediante _MSIX app attach_

## MSIX app attach

[**MSIX app attach**](https://docs.microsoft.com/en-us/azure/virtual-desktop/what-is-app-attach) es un método para desplegar aplicaciones a máquinas físicas y virtuales. Esta tecnología permite mantener una **imagen pequeña** e integrar las aplicaciones desde un [**contenedor MSIX**](https://docs.microsoft.com/en-us/windows/msix/desktop/desktop-to-uwp-behind-the-scenes).

{: .box-note}
**Nota**: Es imprescindible tener un **certificado** para [firmar las aplicaciones](https://docs.microsoft.com/en-us/windows/msix/package/signing-package-overview). Este certificado puede ser emitido por una _Root CA_ interna o pública pero es necesario que el sistema operativo la reconozca para que se pueda usar la aplicación.

En un futuro no muy lejano, se espera que los fabricantes proporcionen sus aplicaciones directamente en un paquete MSIX de tal forma que se pueda usar directamente. También será posible convertirlo a formato VHD, VHDX o CIM para poder **cargar** la aplicación ([_app attach_](https://docs.microsoft.com/en-us/azure/virtual-desktop/app-attach)).

{: .box-note}
**Nota**: Para poder realizar esta carga de aplicaciones MSIX es necesario utilizar Windows 10 2004 o superior.

### Creación de un paquete MSIX

Para probar lo explicado por Christiaan, he creado un paquete MSIX con una aplicación "clásica" [Notepad++](https://notepad-plus-plus.org/
) mediante los pasos siguientes:

* Instalar la herramienta [**MSIX Packaging Tool**](https://www.microsoft.com/en-us/p/msix-packaging-tool/9n5lw3jbcxkf) desde la Microsoft Store

![MSIX Packagint Tool][1]

* Descargar [Notepad++ 7.9.1](https://github.com/notepad-plus-plus/notepad-plus-plus/releases/download/v7.9.1/npp.7.9.1.Installer.x64.exe)

{: .box-warning}
**Atención**: Se recomienda que _Windows Update_, _Windows Search_ y _SMS Hosts_ estén deshabilitados para que no interfieran en la captura del paquete. Christiaan ejecuta los siguientes comandos para preparar la máquina:

```
# Código de Christiaan
reg.exe add HKLM\Software\Policies\Microsoft\WindowsStore /v AutoDownload /t REG_DWORD /d 0 /f
schtasks.exe /Change /Tn "\Microsoft\Windows\WindowsUpdate\Scheduled Start" /Disable
reg.exe add HKCU\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v PreInstalledAppsEnabled /t REG_DWORD /d 0 /f
reg.exe add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ContentDeliveryManager\Debug /v ContentDeliveryAllowedOverride /t REG_DWORD /d 0x2 /f
```
<p></p>

* Ejecutar la herramienta MSIX Packaging Tool
* Seleccionar la opción **Crear paquete de aplicación**
* Seleccionar la opción **Crear paquete en este equipo** (también se puede crear en una máquina remota o en una máquina virtual)

La herramienta instalará el controlador para empaquetar aplicaciones y comprobará que los servicios indicados anteriormente estén deshabilitados

* Seleccionar el instalador de la aplicación `npp.7.9.1.Installer.x64` (se podrían usar argumentos para realizar la instalación desatendida)
* Seleccionar la opción **Firmar con un certificado (.pfx)**

**Nota**: Para realizar esta prueba, se ha creado un certificado auto-firmado usando el _cmdlet_ de PowerShell `New-SelfSignedCertificate`. Después se ha exportado a un fichero `*.pfx` y se ha importado en el **almacén de certificados** (Certificados - Usuario actual > Personas de confianza > Certificados).

```
New-SelfSignedCertificate -Type Custom -Subject "CN=UPC" -KeyUsage DigitalSignature -KeyAlgorithm RSA -KeyLength 2048 -CertStoreLocation "Cert:\LocalMachine\My"
$Password = ConvertTo-SecureString -String "P@ssw0rd!" -AsPlainText -Force
Get-ChildItem -Path Cert:\LocalMachine\My | Export-PfxCertificate -FilePath C:\UPC-self-signed.pfx -Password $Password
Import-PfxCertificate -FilePath C:\UPC-self-signed.pfx -CertStoreLocation Cert:\LocalMachine\TrustedPeople\ -Password $Password
```
<p></p>

* Seleccionar el certificado generado anteriormente `C:\mypfx.pfx` y escribir su contraseña
* Completar la información del paquete:
  * Nombre del paquete: NotepadPlusPlus
  * Nombre para mostrar del paquete: Notepad++
  * Nombre del editor: CN=UPC (proviene del certificado)
  * Nombre para mostrar del editor: UPC
  * Versión: 7.9.1.0
  * Descripción del paquete: Editor de código fuente gratuito
* Instalar la aplicación como se haría normalmente (hay que **deshabilitar** cualquier opción de **actualización automática** de la misma)
  * Installer Language: Español
  * Bienvenido al asistente de instalación > Siguiente
  * Acuerdo de licencia > Acepto
  * Directorio de destino: `C:\Program Files\Notepad++` > Siguiente
  * Selección de componentes: Personalizada
    * [X] Auto-completion files
    * [X] Function List Files
    * [X] Plugins
    * [ ] Auto-Updater
    * [ ] Plugins Admin
    * [ ] Localization
      * [X] Catalan
      * [X] English
      * [X] Spanish
    * [X] Themes
    * [X] Context Menu Entry
  * [ ] Create Shortcut on Desktop > Instalar
  * [X] Ejecutar Notepad++ v7.9.1 > Terminar

{: .box-note}
**Nota**: La ejecución de la aplicación sirve para comprobar que funciona correctamente y/o realizar las configuraciones necesarias. Si es necesario reiniciar el ordenador después de la instalación, hay que hacerlo antes de continuar con el asistente de la herramienta MSIX Packaging Tool.

* Administrar tareas de primer inicio para indicar el punto de entrada de la aplicación:
  * Nombre: `notepad++.exe`
  * Ruta de acceso al archivo: `C:\Program Files\Notepad++\notepad++.exe`
* Finalizar el asistente seleccionando la opción `Sí, continuar' (si hubiera que capturar aplicaciones adicionales se seleccionaría 'No, no he terminado')
* Comprobar si hay servicios en el paquete (en este caso no hay ninguno)
* Crear el paquete

El resultado es un fichero `NotepadPlusPlus_7.9.1.0_x64__hd37mr70wnx6p.msix` en la ubicación que se le haya indicado y el registro de la conversión en el directorio `%LOCALAPPDATA%\Packages\Microsoft.MsixPackagingTool_8wekyb3d8bbwe\LocalState\DiagOutputDir\Logs`.

![Paquete MSIX creado correctamente][2]

## Despliegue de la aplicación MSIX

Para probar este paquete en otra máquina con Windows 10 únicamente es necesario hacer lo siguiente:

* Importar el certificado auto-firmado para que se reconozca la firma (no haría falta si se hubiese firmado con un certificado emitido por una CA renocida por Windows)
* Doble clic encima del fichero MSIX para instalarla (se puede hacer desde un usuario que no sea administrador ;-):

![Doble clic para instalar el paquete MSIX][3]

* Después de instalar el paquete y ejecutar la aplicación, se puede comprobar que ésta no aparece en `Panel de control\Programas\Programas y características` ni tampoco en `C:\Program Files\Notepad++`. Al tratarse de una aplicación UWP "moderna" se instala bajo `C:\Program Files\WindowsApps`.

![Notepad++ en aplicaciones universales][4]

{: .box-note}
**Nota**: Es posible realizar todo el proceso de empaquetado MSIX desde la [línea de comandos](https://docs.microsoft.com/en-us/windows/msix/package/manual-packaging-root). También es posible instalar/instalar el paquete MSIX usando PowerShell.

* Instalar un paquete MSIX:

```
Add-AppxPackage -Path .\NotepadPlusPlus_7.9.1.0_x64__hd37mr70wnx6p.msix
```
<p></p>

* Obtener información sobre el paquete `NotepadPlusPlus`:

```
PS C:\Users\Software\Desktop> Get-AppxPackage -Name NotepadPlusPlus


Name              : NotepadPlusPlus
Publisher         : CN=UPC
Architecture      : X64
ResourceId        :
Version           : 7.9.1.0
PackageFullName   : NotepadPlusPlus_7.9.1.0_x64__hd37mr70wnx6p
InstallLocation   : C:\Program Files\WindowsApps\NotepadPlusPlus_7.9.1.0_x64__hd37mr70wnx6p
IsFramework       : False
PackageFamilyName : NotepadPlusPlus_hd37mr70wnx6p
PublisherId       : hd37mr70wnx6p
IsResourcePackage : False
IsBundle          : False
IsDevelopmentMode : False
NonRemovable      : False
IsPartiallyStaged : False
SignatureKind     : Developer
Status            : Ok
```
<p></p>

* Desinstalar un paquete:

```
Remove-AppxPackage -AllUsers -Package NotepadPlusPlus_7.9.1.0_x64__hd37mr70wnx6p
```

## Contenedores VHD

Christiaan explica que, cuando se utiliza _app attach_ en Windows Virtual Desktop, se puede utilizar dos formatos de contenedor: **VHD** y **CIM**. Los ficheros `*.cim` están asociados a [**Composite Image File System (CimFS)**](https://docs.microsoft.com/en-us/azure/virtual-desktop/app-attach-glossary#cim).

La principal ventaja de estos ficheros es que se pueden montar/desmontar más rapidamente que los ficheros VHD. Además consumen menos disco y memoria.

La creación de un **contenedor VHD** que contenga el paquete MSIX para poder usarlo en _app attach_ (montándolo como si de un simple disco duro se tratara) se puede realizar de varias maneras:

* De forma manual, mediante la herramienta [**MSIX Manager Tool**](https://aka.ms/msixmgr)  (Microsoft):
  * Descargar la herramienta
  * Extraer el fichero ZIP en un directorio, p.ej. `C:\MSIXMgr`
  * Copiar al mismo directorio el paquete MSIX que se ha generado anteriormente
  * Ejecutar los siguientes comandos en PowerShell

```
# Creación del VHD (hay que ajustar el tamaño según la aplicación)
# Es necesario tener instalado Hyper-V para tener acceso al cmdlet `New-VHD`
New-VHD -SizeBytes 1024MB -Path C:\MSIXMgr\notepadplusplus.vhd -Dynamic -Confirm:$false

# Montar el contenedor
$vhd = Mount-VHD C:\MSIXMgr\notepadplusplus.vhd -Passthru

# Inicializar el disco
$disk = Initialize-Disk -Passthru -Number $vhd.Number

# Crear la tabla de particiones (se asignará una letra)
$partition = New-Partition -AssignDriveLetter -UseMaximumSize -DiskNumber $disk.Number

# Formatear el contenedor
Format-Volume -FileSystem NTFS -Confirm:$false -DriveLetter $partition.DriveLetter -Force

# Crear una carpeta y extraer el fichero MSIX en ella
New-Item -Type Directory "$($partition.DriveLetter):\notepadplusplus"
C:\MSIXMgr\msixmgr.exe -Unpack -packagePath C:\MSIXMgr\Notepadplusplus_1.0.0.0_x64__h91ms92gdsmmt.msix -destination "$($partition.DriveLetter):\notepadplusplus" -applyacls

# Desmontar el contenedor
Dismount-VHD C:\MSIXMgr\notepadplusplus.vhd -Passthru
```
<p></p>

* De forma automática, mediante un par de herramientas gratuitas, que evitan la necesidad de disponer de los módulos PowerShell de Hyper-V:
  * **AppVentiX** (Community Tool)
  * **MSIX Hero**

{: .box-warning}
**Atención**: La creación del contenedor en VHD no se ha probado. Tampoco ninguna de las herramientas indicadas anteriormente. Una vez creado el contenedor VHD faltaría asociarlo con una máquina física o virtual para que la aplicación apareciera instalada (_app attach_).

## Referencias

* Video [MSIX Walkthrough](https://www.microsoft.com/es-es/videoplayer/embed/RE3ig2l?autoCaptions=es-es) de [John Vintzel](https://twitter.com/jvintzel) (Principal Program Manager MSIX)
* Video [MSIX in the Enterprise](https://www.youtube.com/watch?v=orh87lo6VKY) de [John Vintzel](https://twitter.com/jvintzel) (Principal Program Manager MSIX)
* Libro [MSIX Packaging Fundamentals](https://www.advancedinstaller.com/downloads/MSIX-Packaging-Fundamentals-free-ebook.pdf) [PDF, 202pág.] de [Timothy Mangan](https://twitter.com/timothymangan), [Kevin Kaminski](https://twitter.com/kkaminsk) y [Bogdan Mitrache](https://twitter.com/BogdanMitrache)
* Herramienta [MSIX Manager Tool](https://aka.ms/msixmgr), herramienta de Microsoft para gestionar los paquetes MSIX
* Herramienta [AppVentiX](https://appventix.com/), anteriormente conocida como App-V Scheduler, sirve para gestionar el ciclo de vida de App-V y MSIX
* Herramienta [MSIX Hero](https://msixhero.net/), un conjunto de herramientas para gestionar los paquetes MSIX
* Blog [A Deep Dive into MSIX App Attach – Windows Virtual Desktop](https://ryanmangansitblog.com/2020/08/20/a-deep-dive-into-msix-app-attach-windows-virtual-desktop/) de [Ryan Mangan](https://twitter.com/Rymangan)

[1]: /assets/img/blog/2020-11-21_image_1.png "MSIX Packagint Tool"
[2]: /assets/img/blog/2020-11-21_image_2.png "Paquete MSIX creado correctamente"
[3]: /assets/img/blog/2020-11-21_image_3.png "Doble clic para instalar el paquete MSIX"
[4]: /assets/img/blog/2020-11-21_image_4.png "Notepad++ en aplicaciones universales"
