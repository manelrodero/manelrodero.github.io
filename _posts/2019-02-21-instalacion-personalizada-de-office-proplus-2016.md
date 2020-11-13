---
layout: post
title: Instalación personalizada de Office ProPlus 2016
date: 2019-02-21 22:26:59.000000000 +01:00
published: true
tags:
- Windows
- Office
- OSDUpdate
author:
  display_name: Manel Rodero
---

En un entorno universitario como el nuestro, es bastante habitual que se necesite instalar aplicaciones de forma desatendida (silenciosa) o personalizada.

En este post se explicará cómo hacerlo con **[Microsoft Office Professional Plus 2016][1]** (se necesita una licencia por volumen y activación mediante KMS) ya que el procedimiento es algo diferente al de **[Microsoft Office365 ProPlus][2]** (se trata de una suscripción).

## Descarga del fichero ISO

La descarga del fichero ISO de Microsoft Office ProPlus 2016 x64 se puede realizar desde el portal de licencias [VLSC][3] de Microsoft o desde el portal de Distribución de Software de la Universidad.

## Preparación del instalador

Una vez descargado el fichero ISO hay que descomprimirlo en un directorio desde el cual se realizará la instalación. Este directorio contendrá también la configuración a aplicar y las actualizaciones del producto. De esta manera, una vez instalado no será necesario buscarlas en Microsoft Update o un servidor WSUS.

* Crear una carpeta compartida en un servidor (por ejemplo, `G:\MICROSOFT\OFFICE\2016\x64`)
* Montar el fichero ISO haciendo doble click en el Explorador de Archivos (Windows asignará una letra de unidad, por ejemplo la D:)
* Copiar todo el contenido del CD en la carpeta creada anteriormente en el servidor

La copia de los ficheros se puede realizar mediante el Explorador de Archivos o mediante el comando robocopy.exe para preservar las fechas de los ficheros y directorios:
    
    
    robocopy.exe D:\ G:\MICROSOFT\OFFICE\2016\x64 /E /XJ /COPY:DAT /DCOPY:DAT /LOG:G:\LOGS\Office2016.log
    

> Un "truco" para minimizar el espacio ocupado por Office, Project y Visio es copiarlos en la misma carpeta del servidor ya que comparten muchos de sus ejecutables y componentes.

La copia de los CD de Project y Visio se realizaría mediante el comando robocopy.exe para evitar escribir encima de ficheros existentes:
    
    
    robocopy.exe D: G:\MICROSOFT\OFFICE\2016\x64 /E /XJ /COPY:DAT /DCOPY:DAT /XC /XN /XO /LOG:G:\LOGS\Project2016.log
    robocopy.exe D: G:\MICROSOFT\OFFICE\2016\x64 /E /XJ /COPY:DAT /DCOPY:DAT /XC /XN /XO /LOG:G:\LOGS\Visio2016.log
    

A continuación se descarga la [última versión de la herramienta OCT][4] (incluida junto a las plantillas administrativas de Office 2016), se descomprime y se copia en el directorio "admin" de la carpeta creada en el servidor mediante el comando robocopy.exe para actualizar los ficheros existentes:
    
    
    robocopy.exe C:\DOWNLOADS\ADMX\OFFICE\2016\admin G:\MICROSOFT\OFFICE\2016\x64admin /E /XJ /COPY:DAT /DCOPY:DAT /XC /XO /LOG:G:\LOGS\OCTAdmin.log
    

## Ejecución de Microsoft OCT

La herramienta [Microsoft Office Customization Tool][5] (OCT) permite personalizar la instalación de una edición de Office con licencia por volumen.

> Es muy importante utilizar la OCT correspondiente al producto y arquitectura que se quiere personalizar (por ejemplo, Office 2016 x64, Project 2016 x86, etc.

Para ejecutar la herramienta se utiliza el siguiente comando desde la carpeta creada en el servidor:
    
    
    setup.exe /admin
    

Al ejecutar OCT por primera vez, aparece un diálogo para seleccionar el producto que se quiere personalizar (en este caso se selecciona **Microsoft Office Professional Plus 2016 (64-bit)** ya que no se ha integrado Visio ni Project:

![Selección del producto a personalizar][6]

A continuación aparece la pantalla principal de la herramienta OCT. La parte izquierda de la pantalla permite navegar por las diferentes secciones de la configuración y la parte derecha muestra las opciones disponibles para cada una de ellas:

![Herramienta Microsoft Office Customization Tool (OCT)][7]

## Personalización

La personalización de Microsoft Office 2016 depende de las necesidades de cada entorno (por ejemplo, en el nuestro se ha decidido no instalar Outlook porque se utiliza Mozilla Thunderbird como lector de correo predeterminado) y es posible que sea necesario crear más de una para soportar diferentes tipos de usuario.

A continuación se detallan los cambios que se suelen realizar en nuestro entorno (se han marcado con un * los que pueden resultar más conflictivos o necesitar algún cambio puntual según el tipo de usuario al que van dirigidos):
* Install location and organization name
    * Default installation path: `[ProgramFilesFolder]\Office2016`
    * Organization name: FIB – UPC
* Additional network sources
    * Marcar "Clear existing server list"
* Licensing and user interface
    * Seleccionar "Use KMS client key"
    * Marcar "I accept the terms in the License Agreement"
    * Display level: Basic (para ver el progreso de la instalación; None para ocultarlo)
    * Desmarcar "Completion notice"
    * Marcar "Suppress modal"
    * Marcar "No cancel"
* Remove previous installations
    * Seleccionar "Remove the following earlier versions of Microsoft Office programs"
    * Comprobar que todos los productos se configuran con "Remove all"
* Modify setup properties
    * Añadir propiedad AUTO_ACTIVATE = 1
    * Añadir propiedad HIDEUPDATEUI = True
    * Añadir propiedad SETUP_REBOOT = Never
* Modify user settings
    * Microsoft Office 2016 > Global Options > Customize > Menu animations = Disabled
    * Microsoft Office 2016 > Help > Federated search for help = Disabled
    * Microsoft Office 2016 > Privacy > Trust Center > Sent personal information = Disabled
    * Microsoft Office 2016 > Privacy > Trust Center > Disable opt-in Wizard on first run = Enabled
    * Microsoft Office 2016 > Privacy > Trust Center > Enable Customer Experience Improvement Program = Disabled
    * Microsoft Office 2016 > Privacy > Trust Center > Automatically receive small updates to improve reliability = Disabled
    * Microsoft Office 2016 > Privacy > Trust Center > Send Office Feedback = Disabled
    * Microsoft Office 2016 > Privacy > Trust Center > Allow including screenshot with Office Feedback = Disabled
    * Microsoft Office 2016 > Services > Fax > Disable Internet Fax feature = Enabled
    * Microsoft Office 2016 > Miscellaneous > Suppress recommended settings dialog = Enabled
    * Microsoft Office 2016 > Miscellaneous > Control Blogging > Enabled
    * Microsoft Office 2016 > Miscellaneous > Show OneDrive Sign In = Disabled (*)
    * Microsoft Office 2016 > Miscellaneous > Block signing into Office = Enabled (*)
    * Microsoft Office 2016 > First Run > Disable First Run Movie = Enabled
    * Microsoft Office 2016 > First Run > Disable Office First Run on application boot = Enabled
    * Marcar "Migrate user settings"
* Set feature installation states
    * Seleccionar "Run all from My computer" para todos los componentes
    * Seleccionar "Not available" para OneDrive, Outlook y Skype
* Add registry entries
    * Agregar en `HKCU\Software\Microsoft\Office\16.0\Common\General` la clave "DisableBootToOfficeStart" de tipo REG_SZ con valor **1**
    * Agregar en `HCKU\Software\Microsoft\Office\16.0\FirstRun` la clave "disablemovie" de tipo REG_SZ con valor **1**
    * Agregar en `HKLM\Software\FIB` la clave "OCT Office ProPlus 2016 x64" de tipo REG_SZ con valor **2019-02-21** (es algo opcional para identificar la fecha de la configuración)

![Componentes seleccionados para instalar][8]

La idea principal de la configuración anterior es permitir que la **instalación** se pueda realizar **sin la interacción del usuario** y minimizando o eliminando las preguntas o ventanas que aparecen durante la primera ejecución (por ejemplo la opción para ver un vídeo sobre Office o autenticarse utilizando una cuenta Microsoft).

> Aunque se pueden configurar muchísimas opciones utilizando OCT, es mejor utilizar GPO para aquellas configuraciones que son específicas de la organización y no de la instalación.

Una vez configuradas todas las opciones, se graba el fichero **MSP** que se utilizará posteriormente durante la instalación desatendida. Este fichero se puede grabar en sitios diferentes según cómo se vaya a invocar la instalación:
* **Opción 1**: grabarlo en el directorio "Updates" para que sea seleccionado automáticamente por el instalador de Microsoft Office (setup.exe) sin tener que especificarlo. Para ello, es necesario que sea el primer fichero del directorio por orden alfabético (por ejemplo, `AAA_ConfigOffice2016.msp`). La desventaja de este método es que únicamente puede haber un fichero de configuración.
* **Opción 2**: grabarlo en otra carpeta con cualquier nombre que lo identifique. En este caso, hay que especificarlo como parámetro del instalador setup.exe. La ventaja de este método es que pueden existir diferentes ficheros de configuración para instalar Microsoft Office de manera diferente según el tipo de usuario.

> La configuración de Microsoft **Project** y Microsoft **Visio** se realiza exactamente de la misma manera mediante su herramienta OCT correspondiente.

## Instalación desatendida

La instalación desatendida de Microsoft Office se realiza de forma diferente según se haya grabado el fichero en el directorio "Updates" o fuera de él (lo más habitual es la raíz de la carpeta compartida del servidor).

Si se ha grabado en el directorio "Updates" y no se ha integrado Project ni Visio, únicamente hay que ejecutar:
    
    
    setup.exe
    

Si se ha integrado Project o Visio, es necesario especificar qué producto se quiere instalar. Por ejemplo, para instalar Microsoft Office se utiliza el siguiente comando:
    
    
    setup.exe /config "proplus.wwconfig.xml"
    

Si el fichero de configuración MSP se ha grabado en otro directorio (por ejemplo en la raíz de la carpeta compartida del servidor), hay que especificarlo usando el parámetro **/adminfile**:
    
    
    setup.exe /adminfile "ConfigOffice2016.MSP" /config "proplus.wwconfig.xml"
    

Para instalar Visio y Project se hace exactamente lo mismo:
    
    
    setup.exe /adminfile "ConfigProject2016.MSP" /config "prjstd.wwconfig.xml"
    setup.exe /adminfile "ConfigVisio2016.MSP" /config "vispro.wwconfig.xml"
    

## Actualizaciones

Una vez instalado Microsoft Office, Project o Visio, es necesario instalar las últimas actualizaciones disponibles mediante Microsoft Update o un servidor WSUS.

Dado que este proceso es lento y depende de otros servidores, se suelen descargar previamente las actualizaciones (son ficheros **MSP**) y dejarlas en el directorio "Updates" para que se instalen automáticamente y el producto esté completamente actualizado _out-of-the-box_.

Para descargar estas actualizaciones hay diferentes métodos:

* El más utilizado es obtener estas actualizaciones desde un ordenador que ya tenga Office completamente actualizado. Para ello se utiliza el script [CollectUpdates.vbs][9] proporcionado por Microsoft.
* Usar la herramienta "[WSUS Offline Updater][10]" para descargarlas desde Microsoft.
* Usar la herramienta "[WHDownloader][11]" para descargarlas desde Microsoft.

Pero el método más rápido y efectivo a día de hoy es **utilizar el módulo **[**OSDUpdate**][12]** de **[**David Segura**][13]** para descargar las actualizaciones** e instalarlas posteriormente mediante un script en PowerShell:

![Módulo OSDUpdate][23]

Para instalar el módulo OSDUpdate desde la [PowerShell Gallery][14] se utilizan los siguientes comandos:
    
    
    Install-Module OSDUpdate
    Import-Module OSDUpdate -Force
    

A continuación, se pueden descargar las actualizaciones de Office 2016 (incluyendo las de las herramientas de corrección) utilizando el siguiente comando:
    
    
    Get-OSDUpdateDownloads -CatalogOffice "Office 2016 64-Bit" -OfficeProfile Default -RepositoryRootPath C:\Patches\Office2016
    

![Descarga de las actualizaciones de Office 2016 usando OSDUpdate][15]

Al finalizar la descarga, se obtiene una estructura de directorios que contienen cada una de las actualizaciones en formato MSP, listas para ser instaladas:

![Estructura de directorios con][16] 

La instalación de estas actualizaciones se realiza usando **msiexec.exe** iterando sobre cada uno de los ficheros MSP descargados. David proporciona el script [Install-OSDUpdateOffice.ps1][17] (desde la versión 19.2.22.0) para realizar esta instalación:

Se puede comprobar que el script funciona correctamente examinando las actualizaciones instaladas antes de su ejecución (en este ejemplo, únicamente las correspondientes a Windows):

![Antes del script únicamente hay actualizaciones de Window][18]

Y las actualizaciones instaladas después de su ejecución (en este ejemplo, 104 actualizaciones correspondientes a los diferentes paquetes de Microsoft Office 2016):

![Después del script ya aparecen las actualizaciones de Microsoft Office 2016][19]

La comunidad OSD está encantada con este nuevo módulo de [David Segura][13] y otros como [Sune Thomsen][20] han empezado a [utilizarlo y ampliarlo][21]:

![Script Update-Office][22]

Y tú ¿A qué esperas para automatizar tu instalación de Microsoft Office? ;-)

[1]: https://products.office.com/en-us/business/microsoft-office-volume-licensing-suites-comparison
[2]: https://products.office.com/en-us/academic/compare-office-365-education-plans
[3]: https://www.microsoft.com/Licensing/servicecenter/default.aspx
[4]: https://www.manelrodero.com/blog/office#admx
[5]: https://docs.microsoft.com/en-us/DeployOffice/oct/oct-2016-help-overview
[6]: /assets/img/blog/2019-02-21_image_1.png "Selección del producto a personalizar"
[7]: /assets/img/blog/2019-02-21_image_2.png "Herramienta Microsoft Office Customization Tool (OCT)"
[8]: /assets/img/blog/2019-02-21_image_3.png "Componentes seleccionados para instalar"
[9]: https://docs.microsoft.com/en-us/previous-versions/office/office-2013-resource-kit/cc178995(v%3doffice.15)#testing-and-verifying-the-windows-installer-msp-files
[10]: http://www.wsusoffline.net/
[11]: https://forums.mydigitallife.net/threads/whdownloader-support-and-chat.44645/
[12]: https://www.osdeploy.com/osdupdate/home
[13]: https://twitter.com/SeguraOSD
[14]: https://www.powershellgallery.com/packages/OSDUpdate
[15]: /assets/img/blog/2019-02-21_image_5.png "Descarga de las actualizaciones de Office 2016 usando OSDUpdate"
[16]: /assets/img/blog/2019-02-21_image_6.png "Estructura de directorios con"
[17]: https://twitter.com/SeguraOSD/status/1099062865473036289
[18]: /assets/img/blog/2019-02-21_image_8.png "Antes del script únicamente hay actualizaciones de Window"
[19]: /assets/img/blog/2019-02-21_image_9.png "Después del script ya aparecen las actualizaciones de Microsoft Office 2016"
[20]: https://twitter.com/SuneThomsenDK
[21]: https://twitter.com/SuneThomsenDK/status/1101267406532546561
[22]: /assets/img/blog/2019-02-21_image_10.png "Script Update-Office"
[23]: /assets/img/blog/2019-02-21_image_4.png "Módulo OSDUpdate"
