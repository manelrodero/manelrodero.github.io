---
layout : post
blog-width: true
title: 'Crear un Lab de Configuration Manager usando Hydration Kit'
date: '2022-07-01 17:17:57'
readtime: true
published: true
tags:
- Configuration Manager
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2022-07-01_cover.png"
thumbnail-img: ""
---

Hace unos meses, expliqué cómo tener un laboratorio de [**Microsoft Endpoint Configuration Manager**](https://docs.microsoft.com/en-us/mem/configmgr/) (MEMCM) utilizando los [_Deployment Lab Kit_](la-forma-mas-rapida-para-probar-configuration-manager) de Microsoft.

La ventaja principal de estos kits es la rapidez de instalación (básicamente es descargar el instalador y ejecutarlo para que se despliguen y se pongan en marcha las máquinas virtuales del laboratorio). Las desventajas principales son que los productos incluidos (Windows Server, SQL Server, etc.) tienen una fecha de caducidad y que el instalador únicamente funciona con Hyper-V, el hipervisor de Microsoft.

En este post se explica cómo crear un laboratorio de Configuration Manager usando el [**Hydration Kit**](https://www.deploymentresearch.com/hydration-kit-for-windows-server-2022-sql-server-2019-and-configmgr-current-branch/) de [Johan Arwidmark](https://twitter.com/jarwidmark). Este método es mucho más flexible, ya que permiti usar versiones de los productos que no tengan caducidad y crear las máquinas virtuales en cualquier hipervisor que permita iniciarlas desde un fichero `*.iso`.

# Requisitos

Para poder utilizar este kit es necesario disponer de:

* Conectividad a Internet para descargar los diferentes componentes
* Una servidor con [**Microsoft Deployment Toolkit**](instalacion-de-mdt-8456) (MDT)

> **Nota**: El _kit_ contiene un _share_ de MDT (llamado **CMLab**) con diferentes _Task Sequence_ para instalar cada una de la máquinas virtuales del laboratorio (DC01, CM01, etc.).

# Servidor MDT

En este caso hemos usado una [instalación de MDT](instalacion-de-mdt-8456), ya existente en nuestro entorno que tiene las siguientes características:

* Máquina virtual con Windows Server 2016 Build 1607
* MDT 8456 con Hotfix KB4564442
* ADK for Windows 11 version 21H2 + WinPE add-on
* Hipervisor VMware ESXi 6.7 con una red LAN `10.0.0.0/24` para las VM
* Router [pfSense](https://www.pfsense.org/) que proporciona DHCP y salida a Internet mediante NAT

# **_Hydration Kit_**

El _Hydration Kit_ para **Windows Server 2022**, **SQL Server 2019** y **Configuration Manager _Current Branch_** se puede descargar desde el [repositorio de GitHub de Johan Arwidmark](https://github.com/DeploymentResearch/HydrationKitWS2022) y su uso es bastante sencillo:

1. Descargar el _software_ necesario (Windows Server, SQL Server, Configuration Manager, etc.)
2. Instalar el _Hydration Kit_ en el _share_ `CMLab` de MDT
3. Copiar el _software_ descargado a las ubicaciones correctas del _share_ `CMLab`
4. Crear el fichero `*.iso` con el **Bootable Hydration Kit** (`MDT Offline Media`)
5. Crear y desplegar las máquinas virtuales del laboratorio en un hipervisor

# Paso 1: Descargar software

Aunque la descarga de software se puede realizar desde cualquier máquina Windows (cliente o servidor), en este caso se relizará en el mismo servidor MDT para facilitar la copia al _share_ `CMLab`.

El script [`Download-SoftwareForHydrationKit.ps1`](https://gist.github.com/manelrodero/4c58e346fbc18e8ae25e9ecb133e4e74) permite descargar de forma automática todo el software mencionado en este paso, incluyendo Windows Server, SQL Server y Configuration Manager desde **Microsoft Evaluation Center**.

```PowerShell
# Para utilizar versiones de evaluación
.\Download-SoftwareForHydrationKit.ps1 -HydrationKitBase "D:\HydrationKitBase"

# Para utilizar versiones con licencia
.\Download-SoftwareForHydrationKit.ps1 -HydrationKitBase "D:\HydrationKitBase" -UseEvaluation $false
```

El script creará una estructura de directorios bajo la carpeta indicada como parámetro del mismo (en este caso `D:\HydrationKitBase`):

```
D:\HydrationKitBase
├───Downloads           (Software y otros componentes descargados de forma automática)
├───Setup               (Componentes listos para ser añadidos al share 'CMLab')
└───Temp                (Directorio temporal)
```

A continuación se indica todo el software necesario para el _Hydration Kit_:

* **Hydration Kit** ([`HydrationKitWS2022.zip`](https://github.com/DeploymentResearch/HydrationKitWS2022/archive/refs/heads/main.zip))
* [Microsoft Deployment Toolkit](https://docs.microsoft.com/en-us/mem/configmgr/mdt) (MDT):
  * [MDT 8456 x64](https://www.microsoft.com/en-us/download/details.aspx?id=54259)
  * [MDT 8456 Hotfix KB4564442](https://support.microsoft.com/help/4564442)
* [BGInfo](https://docs.microsoft.com/en-us/sysinternals/downloads/bginfo)
* [Windows Server 2022 Standard x64](https://docs.microsoft.com/windows-server/), [SQL Server 2019 Standard x64](https://docs.microsoft.com/en-us/sql/) y [Configuration Manager Current Branch 2203](https://docs.microsoft.com/en-us/mem/configmgr/) usando alguna de estas opciones:
  1. [Microsoft Developer Network](https://visualstudio.microsoft.com/msdn-platforms/) (MSDN)
  2. [Volume Licensing Software Center](https://www.microsoft.com/licensing/servicecenter/default.aspx) (VLSC)
  3. [Microsoft Evaluation Center](https://www.microsoft.com/en-us/evalcenter/)

> **Nota**: Para obtener el fichero `*.iso` de "SQL Server 2019 Standard", si se ha descargado la versión de evaluación, hay que ejecutar el instalador web `SQL2019-SSEI-Eval.exe /Action=Download /MediaPath="D:\HydrationKitBase\Downloads" /MediaType=ISO /Quiet`.

* [SQL Server 2019 Cumulative Update (CU)](https://www.microsoft.com/en-us/download/details.aspx?id=100809)

> **Nota**: Configuration Manager requiere únicamente el CU 5, pero es recomendable utilizar la última actualización disponible ([CU 16 KB5011644](http://support.microsoft.com/help/5011644) a fecha de publicación de este artículo).

* [SQL Server 2019 Reporting Services](https://www.microsoft.com/en-us/download/details.aspx?id=100122)
* [SQL Server Management Studio](https://aka.ms/ssmsfullsetup)
* Pre-requisitos de Configuration Manager

> **Nota**: Para descargar los pre-requisitos hay que ejecutar la aplicación `SMSSETUP\BIN\X64\setupdl.exe` desde los ficheros de instalación de Configuration Manager y proporcionar una carpeta para la descarga.

* [Windows ADK for Windows 11](https://docs.microsoft.com/en-us/windows-hardware/get-started/adk-install) + [WinPE add-on](https://docs.microsoft.com/en-us/windows-hardware/get-started/adk-install)

> **Nota**: Hay que descargar la versión **21H2** desde [`Other ADK downloads`](https://docs.microsoft.com/en-us/windows-hardware/get-started/adk-install#other-adk-downloads). La versión 22H2, liberada en Mayo de 2022, aún no está soportada por Configuration Manager.

> **Nota**: Para descargar el ADK _standalone_ completo, hay que ejecutar `adksetup.exe /layout "D:\HydrationKitBase\Setup\Windows ADK 11" /quiet /ceip off`. Y lo mismo para el WinPE add-on, `adkwinpesetup.exe /layout "D:\HydrationKitBase\Setup\Windows ADK 11 WinPE add-on" /quiet /ceip off`.

* [SQL Server 2019 Express](https://www.microsoft.com/en-us/Download/details.aspx?id=101064), como componente opcinal para usarlo en la instalación del servidor `MDT01`

> **Nota**: Para obtener el instalador de "SQL Server 2019 Express Core" hay que ejecutar el instalador web `SQL2019-SSEI-Expr.exe /Action=Download /MediaPath="D:\HydrationKitBase\Downloads" /MediaType=Core /Quiet`.

## Evaluación vs Licencia

El script descarga las versiones de evaluación de Windows Server, SQL Server y Configuration Manager desde **Microsoft Evaluation Center**.

Aunque estas versiones son completamente funcionales durante 180 días, si se quiere utilizar las versiones con licencia, será necesario descargar los ficheros `*.iso` desde **VLSC** o **MSDN** de forma manual y dejarlos en `D:\HydrationKitBase\Downloads`.

<!-- https://www.tablesgenerator.com/markdown_tables -->

|                               | Evaluación (script)                  | VLSC (manual)                                                                     |
|-------------------------------|--------------------------------------|-----------------------------------------------------------------------------------|
| Windows Server 2022           | <sub>`WindowsServer2022_x64_en-us_Eval.iso`</sub> | <sub>`SW_DVD9_Win_Server_STD_CORE_2022_2108.1_64Bit_English_DC_STD_MLF_X22-82986.iso`</sub>    |
| SQL Server 2019               | <sub>`SQLServer2019-x64-ENU.iso`</sub>            | <sub>`SW_DVD9_NTRL_SQL_Svr_Standard_Edtn_2019Dec2019_64Bit_English_OEM_VL_X22-22109.ISO`</sub> |
| Configuration Manager CB 2023 | <sub>`MEM_ConfigManager_2203_Eval.exe`</sub>      | <sub>`SW_DVD5_MEM_ConfigMgrClt_ML_2203_MultiLang_ConfMgr_MLF_X23-12967.ISO`</sub>              |

# Paso 2: Instalar _Hydration Kit_

## Pre-requisitos

Para poder usar _Hydration Kit_ es necesario disponer de un servidor MDT 8456 con Windows ADK for Windows 11 21H2.

Los pasos necesarios para [instalar MDT 8456](instalacion-de-mdt-8456) son los siguientes:

> **Nota**: En este caso, no ha sido necesario realizar esta instalación porque ya se dispone de un servidor, tal como se comentó anteriormente.

* Instalar Windows ADK for Windows 11
* Instalar el WinPE add-on del ADK anterior
* Instalar MDT 8456 x64
* Instalar MDT 8456 HotFix KB4564442 (sustitución del fichero `Microsoft.BDD.Utility.dll` de la instalación de MDT):

```PowerShell
$HydrationKitBase = "D:\HydrationKitBase"
$Source = "$HydrationKitBase\Setup\MDT 8456 HotFix"
$Target = "C:\Program Files\Microsoft Deployment Toolkit\Templates\Distribution\Tools"
Copy-Item -Path "$Source\x86\Microsoft.BDD.Utility.dll" -Destination "$Target\x86" -Force
Copy-Item -Path "$Source\x64\Microsoft.BDD.Utility.dll" -Destination "$Target\x64" -Force
```

## Extracción del kit

Para poder instalar el _Hydration Kit_ en el servidor MDT es necesario **descomprimirlo** y, opcionalmente, **adaptarlo** a nuestro entorno (cambiar las direcciones IP, el nombre del dominio, las contraseñas, etc.).

Si se ha utilizado el script [`Download-SoftwareForHydrationKit.ps1`](https://gist.github.com/manelrodero/4c58e346fbc18e8ae25e9ecb133e4e74) para descargar el software necesario de forma automática, el _Hydration Kit_ ya ha sido extraído en el directorio `D:\HydrationKitBase\Setup\HydrationKit`.

Este directorio contiene un par de scripts en PowerShell, la documentación y el directorio `Source` con el contenido del _share_ que se instalará en el servidor MDT:

```
 Directory of D:\HydrationKitBase\Setup\HydrationKit

<DIR>          docs
         1,075 LICENSE
         7,091 New-HydrationKitSetup.ps1
         3,874 New-LabVMsForHyperV.ps1
        26,191 README.md
<DIR>          Source
```

## Adaptación al entorno

La adaptación del _Hydration Kit_ a un entorno concreto se puede hacer de alguna de éstas maneras:

1. A mano siguiendo las intrucciones del artículo "[Customizing the ViaMonstra Hydration Kit](https://www.deploymentresearch.com/customizing-the-viamonstra-hydration-kit/)" de [Johan Arwidmark](https://twitter.com/jarwidmark)
2. Mediante el script [`CustomizeHydrationKit.ps1`](https://github.com/DeploymentResearch/DRFiles/blob/master/Scripts/CustomizeHydrationKit.ps1) de [Mattias Benninge](https://twitter.com/matbg) referenciado en el artículo "[Customizing the Hydration kit, PowerShell style!](https://deploymentresearch.com/customizing-the-hydration-kit-powershell-style-by-mattias-benninge/) (probado únicamente en el [_Hydration Kit_ para Windows Server 2016](https://www.deploymentresearch.com/hydration-kit-for-windows-server-2016-and-configmgr-current-technical-preview-branch/?pid=145819154))
3. A mano usando la opción `Search > Find in Files... > Replace in Files` de **Notepad++**

En este caso, se ha utilizado la opción 3 para buscar en los ficheros `*.hta`, `*.ini`, `*.ps1`, `*.txt`, `*.vbs`, `*.wsf`, `*.xml` del directorio `D:\HydrationKitBase\Setup\HydrationKit` y:

- Reemplazar `192.168.1` por `10.0.0`
  - Configure-CreateADSubnets.ps1 (1)
  - CustomSettings_CM01.ini (3)
  - CustomSettings_DC01.ini (8)
  - CustomSettings_DP01.ini (3)
  - CustomSettings_FS01.ini (3)
  - CustomSettings_MDT01.ini (3)
- Reemplazar `Pacific Standard Time` por `Romance Standard Time`
  - CM01\Unattend.xml (1)
  - DC01\Unattend.xml (1)
  - FS01\Unattend.xml (1)
  - MDT01\Unattend.xml (1)
  - CustomSettings.ini (1)
- Reemplazar `P@ssw0rd`por una nueva contraseña y apuntarla en KeePass/Bitwarden
  - Configure-CreateADStructure.wsf (16)
  - HYDCMRSPConfig.ps1 (1)
  - CustomSettings.ini (1)
  - CustomSettings_CM01.ini (1)
  - CustomSettings_DC01.ini (1)
  - CustomSettings_FS01.ini (1)
  - CustomSettings_MDT01.ini (1)
- Deshabilitar el grupo `Install DHCP` en el fichero `.\Source\Hydration\Control\DC01\ts.xml`
- Comentar las opciones `DHCPxxxxx` en el fichero `.\Source\Media\Control\CustomSettings_DC01.ini`

> **To-Do**: Adaptar el script [`CustomizeHydrationKit.ps1`](https://github.com/DeploymentResearch/DRFiles/blob/master/Scripts/CustomizeHydrationKit.ps1) de [Mattias Benninge](https://twitter.com/matbg) para que funcione con este _Hydration Kit_.

## Instalación del kit

A continuación se instala el _Hydration Kit_ en el servidor MDT (se necesita disponer de 50GB libres) ejecutando el script `New-HydrationKitSetup.ps1` en `D:\HydrationKitBase\Setup\HydrationKit` desde un PowerShell con **permisos de administrador**:

```PowerShell
.\New-HydrationKitSetup.ps1 -Path "C:\Shares\CMLab" -ShareName "CMLab"
```

Si todo funciona correctamente, se creará un _Deployment Share_ llamado **`CMLab`** que apunta al directorio `C:\Shares\CMLab\DS` y un directorio `C:\Shares\CMLab\ISO` donde generar el fichero `*.iso` con el **Bootable Hydration Kit** (`MDT Offline Media`).

# Paso 3: Copiar software al Hydration Kit DS

## Reference WIM

Antes de copiar el software al _Deployment Share_, es necesario ejecutar el script [`Export-WindowsServer2022WIMfromISO.ps1`](https://gist.github.com/manelrodero/b1b8eba118b9d59b1f7b013193a86fb6) para obtener una imagen `*.wim` de Windows Server 2022 con un único índice.

Este script es una adaptación del [script de Johan Arwidmark](https://github.com/DeploymentResearch/DRFiles/blob/master/Scripts/Export-WindowsServer2022WIMfromISO.ps1) para utilizar la estructura de software descargado anteriormente.

```PowerShell
# Para utilizar versiones de evaluación
.\Export-WindowServer2022WIMfromISO.ps1 -HydrationKitBase "D:\HydrationKitBase"

# Para utilizar versiones con licencia
$ISOName = "SW_DVD9_Win_Server_STD_CORE_2022_2108.1_64Bit_English_DC_STD_MLF_X22-82986.iso"
.\Export-WindowServer2022WIMfromISO.ps1 -HydrationKitBase "D:\HydrationKitBase" -UseEvaluation $false -ISOName $ISOName
```

Si todo funciona correctamente, se extraerá el índice correspondiente a la imagen **Windows Server 2022 SERVERSTANDARD** (_Desktop Experience_) a un fichero `REFWS2022-001.wim` en el directorio `D:\HydrationKitBase\Setup\ReferenceWIM`.

Además, se copiará el fichero `microsoft-windows-netfx3-ondemand-package~31bf3856ad364e35~amd64~~.cab` para evitar problemas con la instalación de .NET Framework 3.5 si las máquinas virtuales no tienen conexión a Internet durante la instalación.

## Copiar software al DS

A continuación, se puede ejecutar el script [`Copy-SoftwareToHydrationKitDS.ps1`](https://gist.github.com/manelrodero/13a67fd84aa7c4f4c6004832eacbd126) para copiar al _Deployment Share_ en `C:\Shares\CMLab\DS` todo el sofware descargado anteriormente, de tal forma que que las _Task Sequence_ estén completas y funcionen correctamente.

```PowerShell
# Para utilizar versiones de evaluación
.\Copy-SoftwareToHydrationKitDS.ps1 -HydrationKitBase "D:\HydrationKitBase" -CMLab "C:\Shares\CMLab"

# Para utilizar versiones con licencia
$SQLISOName = "SW_DVD9_NTRL_SQL_Svr_Standard_Edtn_2019Dec2019_64Bit_English_OEM_VL_X22-22109.ISO"
$CMISOName = "SW_DVD5_MEM_ConfigMgrClt_ML_2203_MultiLang_ConfMgr_MLF_X23-12967.ISO"
.\Copy-SoftwareToHydrationKitDS.ps1 -HydrationKitBase "D:\HydrationKitBase" -CMLab "C:\Shares\CMLab" -UseEvaluation $false -SQLISOName $SQLISOName -CMISOName $CMISOName
```

# Paso 4: Crear _Bootable Hydration Kit_

Para generar la `MDT Offline Media` que contiene el _Bootable Hydration Kit_ hay que hacer lo siguiente:

1. Ejecutar **Deployment Workbench**
2. Expandir `Deployment Shares` y después `Hydration Kit ConfigMgr`
3. Expandir el nodo `Advanced Configuration` y seleccionar el nodo `Media`
4. En el panel derecho, hacer click con el botón derecho en el elemento `MEDIA001` y seleccionar **Update Media Content**

> **Nota**: La actualización de medios puede tardar un poco en ejecutarse, un momento perfecto para tomar un café ;-)

Cuando finalice la actualización de medios, se habrá generado un fichero `HydrationCMWS2022.iso` en el directorio `C:\Shares\CMLab\ISO`. Este fichero tendrá un tamaño entre 14-16GB (el tamaño puede variar un poco dependiendo de las versiones de software que se hayan añadido).

# Paso 5: Crear y desplegar VMs

A continuación se pueden crear las máquinas virtuales en el hipervisor seleccionado (en este caso VMware ESXi) y conectarlas al fichero `HydrationCMWS2022.iso` generado en el paso anterior.

> **To-Do**: Utilizar [VMware PowerCLI](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.esxi.install.doc/GUID-F02D0C2D-B226-4908-9E5C-2E783D41FE2D.html) para automatizar la creación de las máquinas virtuales en [VMware ESXi](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.esxi.install.doc/GUID-B2F01BF5-078A-4C7E-B505-5DFFED0B8C38.html).

Si el hipervisor es Hyper-V, se puede utilizar el script `New-LabVMsForHyperV.ps1` del directorio `D:\HydrationKitBase\Setup\HydrationKit` para crear las máquinas virtuales (más información al final del [artículo de Johan Arwidmark](https://www.deploymentresearch.com/hydration-kit-for-windows-server-2022-sql-server-2019-and-configmgr-current-branch/)).

## DC01

La primera máquina que se tiene que desplegar en este laboratorio es la `DC01` que será el **controlador** del dominio `corp.viamonstra.com`.

Esta máquina virtual se creará con las siguientes características:

* Name = `DC01`
* Compatibility = ESXi 6.7 virtual machine
* Guest OS Family = Windows
* Guest OS Version = Microsoft Windows Server 2016 or later (64-bit)
* Storage = `datastore1`
* CPU: 2 cores / 1 socket
* Memory: 2048 MB / Reserve all guest memory (All locked)
* Hard Disk: 100 GB / Thin provisioned
* Network: VLAN correspondiente a la red 10.0.0.0/24
* Datastore ISO file: `datastore1\ISOs\HydrationCMWS2022.iso`

Iniciar la máquina virtual `DC01`. Después de arrancar desde `HydrationCMWS2022.iso`, y después de que WinPE se haya cargado, seleccionar la secuencia de tareas **DC01**.

Esperar hasta que aparezca el mensaje 'Hydration completed...' en la pantalla de la `Final Configuration Utility for MDT`. Pulsar sobre el botón `Close`para dar por finalizada la instalación y configuración del la máquina.

### Post-configuración

A continuación se instalan las **VMware Tools** para proporcionar información detallada sobre la máquina virtual y permitir las operaciones en el SO invitado (p.ej. apagado elegante, reinicio, etc.):

* Abrir una _browser console_ en ESXi
* Seleccionar `Guest OS > Install VMware Tools` desde el menú `Actions`
* Ejecutar `setup64.exe` desde el DVD que se ha montado automáticamente
* Seleccionar la instalación de tipo **Complete**
* Reiniciar la máquina virtual

Finalmente, se desconecta el fichero `*.iso` del _Hydration Kit_ y se deja `DC01` en ejecución mienstras se implementan el resto de máquinas virtuales del laboratorio.

![DC01][1]

## CM01

Una vez que el controlador de dominio (`DC01`) está en funcionamiento, se puede implementar la máquina virtual `CM01`. No hay que olvidar tener `DC01` en ejecución mientras se implementa `CM01`, ya que ésta se unirá al dominio durante la instalación.

La máquina virtual se creará con las siguientes características:

* Name = `CM01`
* Compatibility = ESXi 6.7 virtual machine
* Guest OS Family = Windows
* Guest OS Version = Microsoft Windows Server 2016 or later (64-bit)
* Storage = `datastore1`
* CPU: 4 cores / 1 socket
* Memory: 16384 MB / Reserve all guest memory (All locked)
* Hard Disk: 300 GB / Thin provisioned
* Network: VLAN correspondiente a la red 10.0.0.0/24
* Datastore ISO file: `datastore1\ISOs\HydrationCMWS2022.iso`

Iniciar la máquina virtual `CM01`. Después de arrancar desde `HydrationCMWS2022.iso`, y después de que WinPE se haya cargado, seleccionar la secuencia de tareas **CM01**.

El mensaje 'Hydration completed...' en la pantalla de la `Final Configuration Utility for MDT` puede tardar un buen rato en aparecer ya que esta máquina virtual es bastante pesada de instalar. Pulsar sobre el botón `Close` para dar por finalizada la instalación y configuración del la máquina.

### Post-configuración

Después de implementar `CM01`, hay que asegurarse de que la máquina tenga acceso a Internet (usando el NAT del hipervisor o instalando un router virtual como el usado en este caso, **pfSense**).

Abrir **Configuration Manager Console** y acceder al área de trabajo `Administration`. Allí se selecciona el nodo `Updates and Servicing` y se pulsa el botón `Chek for updates` para instalar las últimas actualizaciones disponibles.

Después de aplicar todas las [actualizaciones](https://docs.microsoft.com/en-us/mem/configmgr/core/servers/manage/updates), generalmente hay que reiniciar el servidor o la consola para que se apliquen correctamente los cambios.

> **Nota**: A fecha de este artículo, hay un HotFix Rollup ([KB14244456](https://docs.microsoft.com/en-us/mem/configmgr/hotfix/2203/14244456)) del 9 de Junio de 2022. Este Rollup incluye el HotFix ([KB14480034](https://docs.microsoft.com/en-us/mem/configmgr/hotfix/2203/14480034)) que solucionaba el problema con los registros de los clientes PKI.

A continuación se instalan las **VMware Tools** siguiendo los mismos pasos que en la máquina anterior y se desconecta el fichero `*.iso` del _Hydration Kit_ de la máquina virtual.

![CM01][2]

## MDT01, DP01 y FS01

Estas máquinas virtuales son opcionales y no son necesarias para probar la mayoría de características de MEMCM (más información al final del [artículo de Johan Arwidmark](https://www.deploymentresearch.com/hydration-kit-for-windows-server-2022-sql-server-2019-and-configmgr-current-branch/)):

* MDT01: Servidor MDT
* DP01: Distribution Point
* FS01: File Server

# Referencias

## Windows Server

* [Windows Server documentation](https://docs.microsoft.com/windows-server/)
* [Windows Server Tech Community](https://techcommunity.microsoft.com/t5/windows-server/ct-p/Windows-Server)
* [Get started with Windows Server](https://docs.microsoft.com/windows-server/get-started/get-started-with-windows-server)
* [Windows Server deployment, configuration, and administration Learn Path](https://docs.microsoft.com/learn/paths/windows-server-deployment-configuration-administration/)

## SQL Server

* [Microsoft SQL documentation](https://docs.microsoft.com/en-us/sql/)
* [SQL Server Tech Community](https://techcommunity.microsoft.com/t5/sql-server/bd-p/SQL_Server)
* [Introducing Microsoft SQL Server 2019](https://clouddamcdnprodep.azureedge.net/gdc/gdcJivzXl/original) (Ebook, Packt, 489 páginas)
* [SQL developer tools](https://www.microsoft.com/en-us/sql-server/developer-tools)

## Configuration Manager

* [Microsoft Endpoint Configuration Manager documentation](https://docs.microsoft.com/en-us/mem/configmgr/)
* [Configuration Manager Tech Community](https://techcommunity.microsoft.com/t5/configuration-manager/bd-p/SystemCenterConfigurationManager)
* [Which branch of Configuration Manager should I use?](https://docs.microsoft.com/mem/configmgr/core/understand/which-branch-should-i-use)
* [Supported configurations for Configuration Manager](https://docs.microsoft.com/mem/configmgr/core/plan-design/configs/supported-configurations)

[1]: /assets/img/blog/2022-07-01_image_1.png "DC01"
[2]: /assets/img/blog/2022-07-01_image_2.png "CM01"