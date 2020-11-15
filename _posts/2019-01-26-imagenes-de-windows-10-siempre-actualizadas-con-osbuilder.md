---
layout: post
blog-width: true
title: Imágenes de Windows 10 siempre actualizadas con OSBuilder
date: 2019-01-26 14:00:13.000000000 +01:00
published: true
tags:
- Windows
- OSD
author:
  display_name: Manel Rodero
---

Todo aquel que se dedica al [despliegue de sistemas operativos][1] o a la [gestión del entorno de trabajo moderno][2], seguro que ha pensado muchas veces lo siguiente:

> "Quisiera poder instalar Windows 10 con la última **[actualización acumulativa][3]** disponible sin tener que esperar a descargarla usando [Windows Update][4], con sólo algunas **[Windows apps][5]** instaladas, sin **[OneDrive][6]** o **[Cortana][7]** y con soporte para más de un **[lenguaje][8]**".

Si tú también lo has pensado, continúa leyendo este tutorial sobre cómo hacerlo de forma muy sencilla con el módulo en PowerShell [OSBuilder][9] de [David Segura][10].

Pero antes, un poco de teoría para situarnos ;-)

La [modificación de una imagen Windows][11] se puede realizar de tres maneras diferentes:

1. **[Online servicing][12]**: se utiliza un ordenador de referencia en modo **Audit** para añadir controladores, aplicaciones y configuraciones. Después, se **generaliza** la imagen y se **captura** para poder aplicarla a otros ordenadores.
2. **[Offline servicing][13]**: se utiliza el comando DISM para cambiar una imagen Windows sin iniciarla. La modificación se realiza **montando** la imagen en un directorio temporal en el que se realizarán los cambios. Cuando éstos se aceptan, la imagen queda modificada y se podrá aplicar a otros ordenadores.
3. **[Windows Setup][14]**: se utiliza un fichero de respuestas y configuración personalizado **unattend.xml** durante la instalación con Windows Setup para realizar las modificaciones finales a la instalación.

La primera opción es lo más parecido al método de creación y mantenimiento de imágenes monolíticas que se solía utilizar con Windows XP o Windows 7. La instalación de drivers, aplicaciones y los cambios en la configuración de la imagen se realizan de forma manual (aunque es posible hacerlo mediante _scripts_) y, por tanto, se pueden producir errores e incoherencias entre diferentes versiones de la imagen.

La tercera opción permite configurar algunos aspectos de Windows 10 mediante el fichero [unattend.xml][15] creado con la herramienta [Windows SIM][16] que se incluye en [Windows ADK][17]. Es una opción que permite automatizar bastante la configuración, pero necesita complementarse con otros métodos para instalar Windows 10 completamente actualizado sin descargar las actualizaciones a posteriori.

Finalmente, la **segunda opción**, usando DISM desde una línea de comandos elevada o desde PowerShell, permite automatizar los cambios en una imagen Windows evitando así los posibles errores que se pudieran producir al realizarlos de forma manual.

## Offline Servicing

Para entender mejor qué trabajo simplifica [OSBuilder][9], es interesante conocer el funcionamiento de la herramienta DISM para modificar una imagen Windows sin necesidad de iniciarla.

![Offline Servicing][18]
Fuente: [Microsoft][11]

Los pasos básicos para modificar una imagen Windows usando DISM son los siguientes:

* Abrir un CMD con permisos de Administrador.
* Montar la imagen mediante la opción **/Mount-Image**.
* (Opcional) Añadir o quitar [controladores][19] (**/Add-Driver** o **/Remove-Driver**).
* (Opcional) Añadir o quitar [paquetes][20] (**/Add-Package** o **/Remove-Package**).
* (Opcional) Añadir o quitar [lenguajes][21] (**/Add-Package** o **/Remove-Package**).
* (Opcional) Añadir las últimas actualizaciones SSU, CU o Flash.
* (Opcional) Reducir el tamaño de la imagen eliminando componentes que se han actualizado (**/Cleanup-Image**).
* (Opcional) Aplicar los cambios en la imagen (**/Commit-Image**).
* Desmontar la imagen mediante la opción **/Dismount-Image**.

Por ejemplo, para modificar la imagen de Windows 10 Enterprise que se encuentra en el fichero sourcesinstall.wim del DVD de instalación de Windows 10 y eliminar el paquete "OpenSSH Client" se haría lo siguiente: 
    
    
    mkdir e:\temp\images
    mkdir e:\temp\mount
    copy f:\sources\install.wim e:\temp\images
    dism /mount-image /imagefile:e:\temp\images\install.wim /index:3 /mountdir:e:\temp\mount
    dism /image:e:\temp\mount /remove-package /packagename:OpenSSH-Client-Package~31bf3856ad364e35~amd64~~10.0.17763.1
    dism /unmount-image /mountdir:e:\temp\mount /commit
    rmdir e:\temp\mount
    

## OSBuilder

[OSBuilder][9] es un módulo de PowerShell que ayuda a realizar Offline Servicing a una imagen del sistema operativo Windows. Al usar un método fuera de línea para configurar el sistema operativo, éste se puede utilizar en despliegues como si fuera la imagen original del DVD. Además, esta imagen se puede utilizar para actualizar un sistema operativo ya desplegado, cosa que no puede hacerse con una imagen capturada.

Los pasos básicos para modificar una imagen Windows usando OSBuilder son los siguientes:

* Importar uno o más SO desde el DVD usando el cmdlet **Import-OSMedia**.
* Actualizar los SO importados usando el cmdlet **Update-OSMedia** (se aplicarán actualizaciones del Setup, Servicing Stack, Cumulative Updates, componentes dinámicos y Adobe Flash).
* Crear una tarea para indicar cómo se quiere modificar la imagen usando el cmdlet **New-OSBuildTask** (se pueden añadir/quitar paquetes, lenguajes, controladores, ficheros extra, ejecutar scripts en PowerShell, etcétera).
* Crear una nueva _build_ del SO usando el cmdlet **New-OSBuild** (utilizará el SO actualizado y aplicará los cambios indicados en la tarea creada anteriormente).

**Nota**: OSBuilder permite realizar muchas más cosas (actualizar y configurar un WinPE, crear ficheros ISO a partir de los SO importados o construidos, crear USB arrancables, etcétera) por lo que se aconseja leer la completa [documentación][9] que [David Segura ][10]actualiza constantemente.

En el siguiente ejemplo se modificará una imagen de **Windows 10 Enterprise 1803 x64 es-ES** para:
* Añadir las últimas actualizaciones disponibles.
* Eliminar las aplicaciones que no se necesitan (por ejemplo las relacionadas con Xbox).
* Agregar soporte para múltiples lenguajes (en-US y ca-ES).
* Deshabilitar características que no se requieren (por ejemplo SMBv1).
* Cambiar la configuración de Windows (por ejemplo deshabilitar Cortana o impedir la instalación de aplicaciones de consumidor como Candy Crush y similares).

### Instalar y configurar el entorno

Desde un PowerShell, elevado como Administrador, se ejecuta el siguiente comando para instalar el módulo desde la [PowerShell Gallery][22]:
    
    
    Install-Module -Name OSBuilder -Scope CurrentUser -Force
    

![Instalación de OSBuilder y listado de los cmdlets disponibles][23]

A continuación es necesario configurar el entorno de OSBuilder (básicamente, será indicar dónde se quiere ubicar el directorio en el que se importarán los SO, las actualizaciones descargadas, etcétera).
    
    
    Get-OSBuilder -CreatePaths -SetPath E:EQUIPSOSBuilder
    

![Creación de la estructura de directorios de OSBuilder en E:EQUIPSOSBuilder][24]

### Importar Sistemas Operativos

Para importar un SO, se monta el DVD en una unidad virtual haciendo doble clic sobre el archivo ISO descargado desde el portal de Distribución de Software, VLSC, Imagine, etcétera.

En este caso se utiliza el fichero ISO `SW_DVD9_Win_Pro_Ent_Edu_N_10_1803_64BIT_Spanish_-4_MLF_X21-87181_June2018.iso` correspondiente a **Windows 10 1803 x64 es-ES**:

![ISO montado en la unidad virtual F:][25]

Cada DVD de Windows 10 incorpora las imágenes de diferentes ediciones del SO. En concreto, este DVD incorpora las siguientes:
    
    
    Index 1: Windows 10 Education  
    Index 2: Windows 10 Education N  
    Index 3: Windows 10 Enterprise  
    Index 4: Windows 10 Enterprise N  
    Index 5: Windows 10 Pro  
    Index 6: Windows 10 Pro N  
    Index 7: Windows 10 Pro Education  
    Index 8: Windows 10 Pro Education N  
    Index 9: Windows 10 Pro for Workstations  
    Index 10: Windows 10 Pro N for Workstations

Se importa únicamente la edición **Enterprise** especificándola mediante el parámetro -EditionId:
    
    
    Import-OSMedia -EditionId Enterprise -SkipGridView
    Get-OSMedia
    

![Lista de SO importados (en este caso Windows 10 1803, Build 17134.112)][26]

### Actualizar Sistemas Operativos

Para poder actualizar un Sistema Operativo importado, es necesario descargar las actualizaciones disponibles para el mismo. Antes de poder descargarlas, es necesario obtener el catálogo con las actualizaciones disponibles:
    
    
    Get-OSBUpdate -UpdateCatalogs
    

![Obtención del catálogo (sin descargar las actualizaciones)][27]

A continuación, aunque OSBuilder permite realizar la descarga de las actualizaciones y la actualización del SO por separado, se utiliza una mejora introducida en las últimas versiones del módulo para realizarlo en un único paso:
    
    
    Get-OSMedia | Update-OSMedia -DownloadUpdates -Execute
    

Es decir, se obtienen los diferentes SO del entorno mediante el cmdlet **Get-OSMedia** y se pasan a través de la _pipeline_ al cmdlet **Update-OSMedia** forzando la descarga de las actualizaciones necesarias para cada uno:

![Update-OSMedia descargando las actualizaciones necesarias para Windows 10 1803 x64][28]

Después un tiempo, variable en función del procesador y de la velocidad del disco duro, finaliza el proceso de actualización del Sistema Operativo, obteniendo una nueva OSMedia:

![OSMedia original (importada desde el DVD) y OSMedia actualizada][29]

### Descargar Lenguajes

Uno de los objetivos de este tutorial es conseguir una imagen Windows configurada con varios lenguajes. En este caso se añadirá soporte para **en-US** (inglés, EE.UU.) y **ca-ES** (catalán). El primero es un Language Pack (LP) completo, mientras que el segundo es un Language Interface Pack (LIP) que se aplica a un lenguaje principal, en nuestro caso es-ES.

Se pueden obtener los LP y LIP desde el [Catálogo de Actualizaciones de Microsoft][30] o desde el DVD correspondiente a los lenguajes disponible también en VLSC, Distribución de Software, etc.

> A partir de Windows 10 1809, los antiguos LIP se distribuyen como Local Experience Packs (LXP) desde la Microsoft Store y desde el DVD correspondiente a los lenguajes.

Además, cada lenguaje dispone de una serie de características adicionales ([Features on Demand][31]) que también se pueden descargar: Fuentes, OCR, reconocimiento de escritura a mano, etc.

Por ejemplo, para descargar el lenguaje en-US y sus características adicionales, se utiliza el cmdlet **Get-OSBUpdate**:
    
    
    Get-OSBUpdate -Download -FilterLP en-US -FilterOS "Windows 10" -FilterOSArch x64 -FilterOSBuild 1803 -Catalog LanguagePack
    Get-OSBUpdate -Download -FilterLP en-US -FilterOS "Windows 10" -FilterOSArch x64 -FilterOSBuild 1803 -Catalog LanguageFeature
    
    

![Descarga del LP en-US y sus características adicionales][32]

Para descargar el LIP ca-ES se utiliza el mismo cmdlet con parámetros diferentes:
    
    
    Get-OSBUpdate -Download -FilterLIP ca-ES -FilterOS "Windows 10" -FilterOSArch x64 -FilterOSBuild 1803 -Catalog LanguageInterfacePack
    Get-OSBUpdate -Download -FilterLIP ca-ES -FilterOS "Windows 10" -FilterOSArch x64 -FilterOSBuild 1803 -Catalog LanguageFeature
    
    

### Configurar Sistema Operativo

Las operaciones realizadas hasta el momento han tratado con Sistemas Operativos sin configurar, es decir, **OSMedia** en la nomenclatura de OSBuilder.

Una imagen del Sistema Operativo que, además de las actualizaciones, incluya otros cambios (añadir los lenguajes descargados, añadir o eliminar características, añadir o eliminar paquetes, etcétera) se denomina **OSBuild**. Esta imagen se construye a partir de una tarea (OSBuildTask) que describe las acciones a realizar:
    
    
    New-OSBuildTask -TaskName "Win10 Ent 1803 x64 es-ES" -DisableWindowsOptionalFeature -RemoveAppxProvisionedPackage -SetAllIntl es-ES
    

Al ejecutar el comando anterior, OSBuilder permitirá seleccionar qué OSMedia se quiere utilizar inicialmente:

![Selección de OSMedia inicial para la OSBuildTask][33]

A continuación se seleccionan las aplicaciones aprovisionadas en la imagen -Appx- que se quieren eliminar (es una [decisión muy personal][34] que dependerá de cada entorno):

![Borrado de aplicaciones aprovisionadas no necesarias en la imagen Windows][35]

Las características que se quieren eliminar (en este caso se elimina el soporte para PowerShell v2 para evitar [problemas de seguridad][36]):

![Eliminando soporte para PowerShell v2][37]

Se añaden los lenguajes (LP en-US y LIP ca-ES) que se habían descargado previamente:

![Selección de LP en-US][39]

![Selección de LIP ca-ES][38]

Se añaden las características asociadas a los lenguajes (en este caso, únicamente las **básicas** que incluye corrección ortográfica, predicción de texto, separación de palabras y guiones):

![Selección de Features on Demand (FOD) para los lenguajes][40]

Después de este proceso, OSBuilder genera un fichero JSON en el directorio Tasks con la definición de la tarea para configurar la imagen Windows ([OSBuild Win10 Ent 1803 x64 es-ES.json][41]).

OSBuilder analiza el directorio Content para permitir la selección de paquetes, scripts y ficheros de configuración que se incluirán en una tarea:

* Local Experience Packs extraídos desde el DVD de Lenguajes (directorio ContentIsoExtract).
* Scripts para modificar los ficheros o el registro de la imagen _offline_ (directorio ContentScripts).
* Ficheros XML de configuración desatendida (directorio ContentUnattend).

![Selección de LPX ca-ES para Windows 10 1809][42] 

![Selección de scripts a ejecutar para modificar una OSBuild][43]

![Selección de fichero Unattend.xml que se aplicará a la imagen Windows][44]  

Finalmente, se ejecuta la tarea para construir la OSBuild mediante el cmdlet **New-OSBuild**. En el siguiente ejemplo, se indica la tarea a ejecutar mediante su nombre, se indica que se quiere generar un fichero ISO y que no se quiere utilizar la OSMedia más actual sino la indicada en la tarea.
    
    
    New-OSBuild -Execute -MediaISO -ByTaskName "Win10 Ent 1803 x64 es-ES" -DontUseNewestMedia
    

Después de un tiempo, variable en función de la velocidad del disco duro y otros parámetros del equipo, se obtiene una OSBuild que se puede utilizar para instalar Windows 10 configurado según se haya indicado en la tarea.

La ventaja principal de OSBuilder es que, una vez definida la tarea, ésta se puede reutilizar para aplicar las actualizaciones acumulativas que aparecen cada mes, la configuración indicada en la misma y generar una nueva OSBuild:
    
    
    New-OSBuild -DownloadUpdates -Execute -ByTaskName "Win10 Ent 1803 x64 es-ES"
    

A partir de la versión [19.1.23 ][45]de OSBuilder se pueden utilizar plantillas (**[Templates][46]**) para definir configuraciones globales o específicas para cada familia de Sistemas Operativos.

El resultado de utilizar el fichero ISO de la OSBuild que se acaba de generar para instalar Windows 10 (por ejemplo en una VM Hyper-V, VMware o VirtualBox) es el siguiente:

![Windows 10 1803 x64 instalado a partir de una OSBuild "customizada"][47]

¿Qué os parece? Interesante, ¿verdad? Pues, ya estáis tardando en instalar y probar el módulo en PowerShell [OSBuilder][9] de [David Segura][10].

**Disclaimer**: los comandos utilizados en este tutorial son los existentes en OSBuilder 19.1.24.0 por lo que te recomiendo leer la documentación actualizada para saber si hay nuevos comandos o funcionalidades que permitan realizar las operaciones de una manera diferente.

[1]: https://docs.microsoft.com/en-us/windows/deployment/
[2]: https://www.microsoft.com/en-us/cloud-platform/modern-management
[3]: https://support.microsoft.com/en-us/help/4464619
[4]: https://support.microsoft.com/en-us/help/12373/windows-update-faq
[5]: https://docs.microsoft.com/en-us/windows/application-management/apps-in-windows-10
[6]: https://docs.microsoft.com/en-us/onedrive/
[7]: https://developer.microsoft.com/en-us/cortana
[8]: https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/localize-windows
[9]: https://www.osdeploy.com/osbuilder/overview
[10]: https://twitter.com/SeguraOSD
[11]: https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/modify-an-image
[12]: https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/audit-mode-overview
[13]: https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/mount-and-modify-a-windows-image-using-dism
[14]: https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/windows-setup-automation-overview
[15]: https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/update-windows-settings-and-scripts-create-your-own-answer-file-sxs
[16]: https://docs.microsoft.com/en-us/windows-hardware/customize/desktop/wsim/windows-system-image-manager-overview-topics
[17]: https://docs.microsoft.com/en-us/windows/deployment/windows-adk-scenarios-for-it-pros
[18]: /assets/img/blog/2019-01-26_image_1.png "Offline Servicing"
[19]: https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/add-and-remove-drivers-to-an-offline-windows-image
[20]: https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/add-or-remove-packages-offline-using-dism
[21]: https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/add-and-remove-language-packs-offline-using-dism
[22]: https://www.powershellgallery.com/packages/OSBuilde
[23]: /assets/img/blog/2019-01-26_image_2.png "Instalación de OSBuilder y listado de los cmdlets disponibles"
[24]: /assets/img/blog/2019-01-26_image_3.png "Creación de la estructura de directorios de OSBuilder en E:EQUIPSOSBuilder"
[25]: /assets/img/blog/2019-01-26_image_4.png "ISO montado en la unidad virtual F:"
[26]: /assets/img/blog/2019-01-26_image_5.png "Lista de SO importados (en este caso Windows 10 1803, Build 17134.112)"
[27]: /assets/img/blog/2019-01-26_image_6.png "Obtención del catálogo (sin descargar las actualizaciones)"
[28]: /assets/img/blog/2019-01-26_image_7.png "Update-OSMedia descargando las actualizaciones necesarias para Windows 10 1803 x64"
[29]: /assets/img/blog/2019-01-26_image_8.png "OSMedia original (importada desde el DVD) y OSMedia actualizada"
[30]: https://www.catalog.update.microsoft.com/Home.aspx
[31]: https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/features-on-demand-language-fod
[32]: /assets/img/blog/2019-01-26_image_9.png "Descarga del LP en-US y sus características adicionales"
[33]: /assets/img/blog/2019-01-26_image_10.png "Selección de OSMedia inicial para la OSBuildTask"
[34]: https://www.vacuumbreather.com/index.php/blog/item/69-windows-10-1803-built-in-apps-what-to-keep
[35]: /assets/img/blog/2019-01-26_image_11.png "Borrado de aplicaciones aprovisionadas no necesarias en la imagen Windows"
[36]: https://adsecurity.org/?p=2921
[37]: /assets/img/blog/2019-01-26_image_12.png "Eliminando soporte para PowerShell v2"
[38]: /assets/img/blog/2019-01-26_image_13.png "Selección de LIP ca-ES"
[39]: /assets/img/blog/2019-01-26_image_14.png "Selección de LIP en-US"
[40]: /assets/img/blog/2019-01-26_image_15.png "Selección de Features on Demand (FOD) para los lenguajes"
[41]: https://gist.github.com/manelrodero/c0a6894891f879df540c4a6d8cce63b5
[42]: /assets/img/blog/2019-01-26_image_16.png "Selección de LPX ca-ES para Windows 10 1809"
[43]: /assets/img/blog/2019-01-26_image_17.png "Selección de scripts a ejecutar para modificar una OSBuild"
[44]: /assets/img/blog/2019-01-26_image_18.png "Selección de fichero Unattend.xml que se aplicará a la imagen Windows"
[45]: https://www.osdeploy.com/osbuilder/release-information
[46]: https://www.osdeploy.com/osbuilder/docs/guides/osbuild-templates
[47]: /assets/img/blog/2019-01-26_image_20.png "Windows 10 1803 x64 instalado a partir de una OSBuild 'customizada'"
