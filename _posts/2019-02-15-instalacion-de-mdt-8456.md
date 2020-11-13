---
layout: post
title: Instalación de MDT 8456
date: 2019-02-15 23:51:59.000000000 +01:00
published: true
tags:
- Windows
- OSD
- MDT
author:
  display_name: Manel Rodero
---

[Microsoft Deployment Toolkit ][1](MDT) es una utilidad para el despliegue de sistemas operativos Windows que se puede instalar en un servidor o en una estación de trabajo. Una de las ventajas de instalarlo en un servidor, por ejemplo Windows Server 2016, es la posibilidad de agregar [Windows Deployment Services][2] (WDS) para facilitar el despliegue remoto de los equipos.

En este post se documenta la instalación de la versión [8456][3] en una estación de trabajo Windows 10 x64. Antes de instalar MDT, es necesario instalar [Windows Assessment and Deployment Kit][4] (Windows ADK). En concreto se ha instalado la versión [1809][5] y su correspondiente [add-on para WinPE][6] (en una descargada separada desde esta versión).

## Instalación de Windows ADK

* Ejecutar adksetup.exe:
* Instalar en C:\Program Files (x86)\Windows Kits\10
    * No enviar datos de uso anónimos acerca de los Windows 10 Kits
    * Aceptar la licencia del producto
* Seleccionar para instalar **como mínimo** los siguientes componentes:
    * Deployment Tools (DISM, SIM, BCDBoot, etc.)
    * Imaging and Configuration Designer (ICD)
    * Configuration Designer
    * User State Migration Tool (USMT)
* Instalar **opcionalmente** los siguientes componentes:
    * Microsoft User Experience Virtualization (UE-V) Template

## Instalación de WinPE add-on

* Ejecutar adkwinpesetup.exe
    * Instalar en C:\Program Files (x86)\Windows Kits\10
    * No enviar datos de uso anónimos acerca de los Windows 10 Kits
    * Aceptar la licencia del producto
* Seleccionar para instalar el siguiente componente:
    * Windows Preinstallation Environment (Windows PE)

## Instalación de MDT

* Ejecutar MicrosoftDeploymentToolkit_x64.msi
    * Setup Wizard -> Next
    * Aceptar los términos de licencia
* Seleccionar para instalar todos los componentes:
    * Microsoft Deployment Toolkit
    * Documents
    * Tools and Templates
* Instalar en C:\Program Files\Microsoft Deployment Toolkit
* No participar en el CEIP (Customer Experience Inprovement Program)

## Creación del _share_

Antes de poder trabajar con MDT hay que realizar una mínima configuración inicial **creando la carpeta compartida** (_Deployment Share_) desde la que se desplegarán los sistemas operativos.

En este ejemplo, se creará la carpeta compartida en la unidad `V:\`. Esta operación se puede realizar mediante la herramienta gráfica **Deployment Workbench** (`.\Bin\DeploymentWorkbench.msc`):
* Deployment share path: `V:\MDT\DeploymentShare`
* Share name: DeploymentShare$
* Deployment share description: MDT Deployment Share
* Options: por defecto (se pueden cambiar a posteriori modificando las **reglas** de la carpeta compartida)

O mediante el siguiente código en PowerShell:
    
    
    New-Item -Path "V:\MDT\DeploymentShare" -ItemType directory
    New-SmbShare -Name "DeploymentShare$" -Path "V:\MDT\DeploymentShare" -FullAccess Administrators
    Import-Module "C:\Program Files\Microsoft Deployment Toolkit\Bin\MicrosoftDeploymentToolkit.psd1"
    New-PSDrive -Name "DS001" -PSProvider "MDTProvider" -Root "V:\MDT\DeploymentShare" -Description "MDT Deployment Share" -NetworkPath "\\PCLYM\DeploymentShare$" -Verbose | Add-MDTPersistentDrive -Verbose
    

Opcionalmente, podría ser interesante [modificar los ACLs de la carpeta][7] para que no sean tan restrictivos, por ejemplo agregando una cuenta con permisos para escribir en la carpeta "Captures":
    
    
    $DeploymentShareDir = "V:\MDT\DeploymentShare"
    icacls.exe $DeploymentShareDir /grant '"MDT_User":(OI)(CI)(RX)'
    icacls.exe $DeploymentShareDir /grant '"Administrators":(OI)(CI)(F)'
    icacls.exe $DeploymentShareDir /grant '"SYSTEM":(OI)(CI)(F)'
    icacls.exe "$DeploymentShareDir\Captures" /grant '"MDT_User":(OI)(CI)(M)'
    
    $DeploymentShare = "DeploymentShare$"
    Grant-SmbShareAccess -Name $DeploymentShare -AccountName "EVERYONE" -AccessRight Change -Force
    Revoke-SmbShareAccess -Name $DeploymentShare -AccountName "CREATOR OWNER" -Force
    

A partir de este momento se puede comenzar a utilizar MDT para:

* Importar los **sistemas operativos** que se quieren desplegar
* Agregar **paquetes** como actualizaciones o lenguajes (no es necesario si el SO ya los incluye porque se ha realizado _offline servicing_ del mismo)
* Agregar **aplicaciones** que se instalarán después de desplegar el SO (por ejemplo, los _runtime de Visual C++_)
* Crear una **secuencia de tareas** (_task sequence_) con los pasos que se quieran realizar durante el despliegue

## Importar sistemas operativos

Esta operación se puede realizar desde la interfaz gráfica, pero es mucho más rápido realizarla mediante un script en PowerShell. En el siguiente ejemplo, se importa un Windows Server 2016 actualizado mediante OSBuilder:
    
    
    Import-Module "C:\Program Files\Microsoft Deployment Toolkit\Bin\MicrosoftDeploymentToolkit.psd1"
    New-PSDrive -Name "DS001" -PSProvider MDTProvider -Root "V:\MDT\DeploymentShare"
    Import-MDTOperatingSystem -Path "DS001:Operating Systems" -SourcePath "V:\OSBuilder\OSMedia\Windows Server 2016 Standard Desktop Experience x64 1607 14393.2791OS" -DestinationFolder "Windows Server 2016 Standard x64" -Verbose
    

Es recomendable renombrar los sistemas operativos que se importan con un nombre más corto. En este caso se renombrará a "Windows Server 2016 Standard x64".

## Crear secuencia de tareas

Se creará una secuencia de tareas con las siguientes características:

* Task sequence ID: WIN2016-X64-001
* Task sequence name: Windows Server 2016 x64
* Template: Standard Client Task Sequence
* Operating system: Windows Server 2016 Standard x64
* No especificar una clave de producto
* Nombre/Organización: LYM/LYM
* Internet Explorer home page: www.google.com
* No especificar el password

El código en PowerShell:
    
    
    Import-Module "C:\Program Files\Microsoft Deployment Toolkit\Bin\MicrosoftDeploymentToolkit.psd1"
    New-PSDrive -Name "DS001" -PSProvider MDTProvider -Root "V:\MDT\DeploymentShare"
    Import-MDTTaskSequence -Path "DS001:Task Sequences" -Name "Windows Server 2016 x64" -Template "Client.xml" -Comments "" -ID "WIN2016-X64-001" -Version "1.0" -OperatingSystemPath "DS001:Operating Systems\Windows Server 2016 Standard x64" -FullName "LYM" -OrgName "LYM" -HomePage "www.google.com" -Verbose
    

En este momento se tendría que modificar la tarea para habilitar o deshabilitar fases, agregar pasos y acciones, etc.

![Propiedades de una secuencia de tareas][8]

## Reglas del _share_

A continuación se configuran las **reglas** de la carpeta mediante la modificación de los ficheros Bootstrap.ini y CustomSettings.ini. Estos ficheros indicarán, por ejemplo, si es necesario capturar la imagen, realizar un sysprep o unir el equipo a un dominio. Se puede encontrar más información en la [Toolkit Reference][9] o en el [listado de variables][10] de una secuencia de tareas.

### Bootstrap.ini

Es el primer fichero que se procesa al arrancar en Windows PE. Este fichero se puede dividir en tres secciones: **Settings**, **Custom** y **Default**. El contenido del fichero al crear la secuencia de tareas anterior es el siguiente:
    
    
    [Settings]  
    Priority=Default
    
    [Default]  
    DeployRoot=\\192.168.1.20\DeploymentShare$

Este fichero se puede utilizar para especificar información que permita distinguir a un cierto ordenador de otro (por ejemplo una dirección MAC), una red de otra (por ejemplo un _gateway_), etc. Las secciones de tipo _custom_ se procesan en orden de aparición y la primera que concuerda es la que se aplica. La sección de valores por defecto se suele utilizar para todo aquello que es común a los diferentes despliegues.

Una posible modificación del fichero de configuración que permitiría cambiar la carpeta compartida en función de la puerta de enlace del equipo sería la siguiente:
    
    
    [Settings]  
    Priority=DefaultGateway,MACAdress,Default
    
    [DefaultGateway]  
    192.168.1.1=UnitedStatesHQ  
    192.168.2.1=SpainHQ
    
    [UnitedStatesHQ]  
    DeployRoot=\\192.168.1.20\DeploymentShare$
    
    [SpainHQ]  
    DeployRoot=\\192.168.2.20\DeploymentShare$  
    KeyboardLocalePE=es-ES
    
    [00:11:22:AA:BB:CC]  
    OSDAdapterCount=1  
    OSDAdapter0EnableDHCP=FALSE  
    OSDAdapter0IPAddressList=192.168.2.101  
    OSDAdapter0SubnetMask=255.255.255.0  
    OSDAdapter0Gateways=192.168.2.1  
    OSDAdapter0DNSServerList=8.8.8.8  
    OSDAdapter0DNSSuffix=LYM.LOCAL  
    

En la sección [Default] se suelen incluir las credenciales para autenticarse en la carpeta compartida (DeployRoot) especificada anteriormente. Además, se puede indicar que hay que saltarse la pantalla de bienvenida de BDD y comenzar con el siguiente script:
    
    
    [Default]  
    SkipBDDWelcome=YES  
    KeyboardLocale=es-ES  
    UserID=MDT_User  
    UserPassword=P@ssw0rd  
    UserDomain=LYM.LOCAL

> Siempre que se realiza un cambio en este fichero, es necesario actualizar la carpeta compartida de despliegue ya que este fichero se graba en la imagen de arranque de Windows PE.

### CustomSettings.ini

Este script controla el orden en que se ejecutarán los scripts de MDT y asigna valores a las variables que permitirán hacer un despliegue LTI o ZTI. También tiene tres secciones: Settings, Custom y Default. El contenido del fichero al crear la secuencia de tareas descrita anteriormente es:
    
    
    [Settings]  
    Priority=Default  
    Properties=MyCustomProperty
    
    [Default]  
    OSInstall=Y  
    SkipCapture=NO  
    SkipAdminPassword=YES  
    SkipProductKey=YES  
    SkipComputerBackup=NO  
    SkipBitLocker=NO

La primera sección [Settings] es la encargada de asignar la prioridad de las secciones que están presentes en el fichero y el orden en que deben ser leídas. Las propiedades son variables que se pueden utilizar durante el despliegue:
    
    
    [Settings]  
    Priority=Init,DefaultGateway,ClientType,Model,Default  
    Properties=MyCustomProperty;BIOSVersion;SMBIOSBIOSVersion;ProductVersion,ComputerSerialNumber

Se suele incluir una sección para inicializar variables (por ejemplo, obtener parte del número de serie para construir el nombre del equipo en el AD):
    
    
    [Init]  
    ComputerSerialNumber=#Right("%SerialNumber%",7)#  
    DeploymentOU=OU=Deployment,OU=Computers  
    SitesOU=OU=Sites,DC=LYM,DC=LOCAL
    
    [DefaultGateway]  
    192.168.1.1=LAX  
    192.168.2.1=BCN
    
    [LAX]  
    Localidad=Los Angeles  
    MachineObjectOU=OU=%DeploymentOU%,OU=%Localidad%,OU=%SitesOU%
    
    [BCN]  
    Localidad=Barcelona  
    MachineObjectOU=OU=%DeploymentOU%,OU=%Localidad%,OU=%SitesOU%  
    

También se suele incluir la sección [ClientType] para realizar acciones diferentes según el tipo de cliente (servidor, estación de trabajo, etc.) obtenido mediante consultas WMI. También se puede especificar un modelo concreto de equipo al que se aplicará una determinada configuración:
    
    
    [ClientType]  
    SubSection=Servidor-%IsServer%
    
    [Servidor-True]  
    TaskSequenceID=WIN2016_X64
    
    [Servidor-False]  
    Subsection=Cliente-%IsDesktop%
    
    [Cliente-True]  
    TaskSequenceID=WIN10_X64
    
    [Cliente-False]  
    Subsection=Portatil-%IsLaptop%
    
    [Precision 7530]  
    DriverGroup001=Windows 10x64%Model%  
    DriverSelectionProfile=nothing  
    DeploymentType=NEWCOMPUTER

> Se puede obtener el modelo de un equipo mediante la siguiente consulta en PowerShell: **(Get-WmiObject -Class:Win32_ComputerSystem).Model**.

La sección [Default] es la más importante ya que, habitualmente, se suelen incluir las opciones que se aplicarán a todos los despliegues. Por ejemplo, la variable **_SMSTSORGNAME** incluye la marca que se incluirá en todos los asistentes y diálogos de MDT, la variable **OSDComputername** será el nombre del equipo, etc.
    
    
    [Default]  
    _SMSTSORGNAME=LYM: %OSDComputername%  
    OSInstall=Y  
    DoNotCreateExtraPartition=YES  
    TimeZoneName=Romance Standard Time  
    AdminPassword=P@ssw0rd  
    OSDComputername=%DefaultGateway%-%ComputerSerialNumber%

También se suele utilizar para añadir la información necesaria para unir el equipo al dominio:
    
    
    ;JoinWorkgroup=WORKGROUP  
    JoinDomain=LYM.LOCAL  
    DomainAdmin=Administrator  
    DomainAdminDomain=LYM.LOCAL  
    DomainAdminPassword=P@ssw0rd  
    MachineObjectOU=OU=Unknown Computers,OU=%SitesOU%  
    DomainErrorRecovery=AUTO  
    Administrators001=LYMUser

Otro ejemplo de fichero CustomSettings.ini más completo podría ser el siguiente. Un equipo "Dell Precision 7530", en la red 192.168.2.0/24 con número de serie 12345678, acabaría llamándose LBCN1324567:
    
    
    [Settings]  
    Priority=Default,ByModel,DefaultGateway,SetMacSettings  
    Properties=SiteCode,tSerialNum,OUContainer,OUSite,PCPrefix
    
    [SetMacSettings]  
    OSDComputerName=%PCPrefix%%SiteCode%%tSerialNum%  
    MachineObjectOU=LDAP://%OUContainer%,%OUSite%,DC=contoso,DC=com
    
    [ByModel]  
    Subsection=%Model%
    
    [Precision 7530]  
    OSInstall=Y  
    PCPrefix=L  
    OUContainer=OU=Laptop,OU=Machines  
    tSerialNum=#Right("%SerialNumber%",7)#
    
    [OptiPlex 7020]  
    OSInstall=Y  
    PCPrefix=D  
    OUContainer=OU=Desktop,OU=Machines  
    tSerialNum=#Right("%SerialNumber%",7)#
    
    [VMWare Virtual Platform]  
    OSInstall=Y  
    PCPrefix=V  
    OUContainer=OU=Virtual,OU=Machines  
    tSerialNum=#Right(Replace(Replace(oEnvironment.Item("SerialNumber")," ",""),"-",""),7)#
    
    [BCN]  
    SiteCode=BCN  
    OUSite=OU=Barcelona  
    TimeZoneName=Romance Standard Time
    
    [DefaultGateway]  
    192.168.2.1=BCN

## Configuración Windows PE

Finalmente se configuran las opciones de **Windows PE** para definir qué arquitectura tendrá la **imagen de arranque** (_boot image_), qué **controladores** se añadirán, etc.

![Propiedades de Windows PE x64][11]

Después de **actualizar la carpeta compartida** se puede proceder a desplegar el sistema operativo utilizando la imagen de arranque generada anteriormente en una VM o en una máquina física.
    
    
    Import-Module "C:\Program Files\Microsoft Deployment Toolkit\Bin\MicrosoftDeploymentToolkit.psd1"
    New-PSDrive -Name "DS001" -PSProvider MDTProvider -Root "V:\MDT\DeploymentShare"
    Update-MDTDeploymentShare -path "DS001:" -Verbose
    

Pero eso queda para otro post ;-)

[1]: https://docs.microsoft.com/en-us/sccm/mdt/
[2]: https://docs.microsoft.com/en-us/windows/desktop/Wds/windows-deployment-services-portal
[3]: https://www.microsoft.com/en-us/download/details.aspx?id=54259
[4]: https://docs.microsoft.com/en-us/windows-hardware/get-started/adk-install
[5]: https://go.microsoft.com/fwlink/?linkid=2026036
[6]: https://go.microsoft.com/fwlink/?linkid=2022233
[7]: https://deploymentresearch.com/Research/Post/654/Building-a-Windows-10-v1709-reference-image-using-MDT
[8]: /assets/img/blog/2019-02-15_image_1.png "Propiedades de una secuencia de tareas"
[9]: https://docs.microsoft.com/en-us/sccm/mdt/toolkit-reference
[10]: https://docs.microsoft.com/en-us/sccm/osd/understand/task-sequence-variables
[11]: /assets/img/blog/2019-02-15_image_2.png "Propiedades de Windows PE x64"
