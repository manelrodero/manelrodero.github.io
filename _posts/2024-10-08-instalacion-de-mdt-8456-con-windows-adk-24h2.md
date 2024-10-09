---
layout : post
blog-width: true
title: 'Instalación de MDT 8456 con Windows ADK 24H2'
date: '2024-10-08 13:01:17'
last-updated: '2024-10-09 22:01:17'
published: true
tags:
- Microsoft Deployment Toolkit
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2024-10-08_cover.png"
thumbnail-img: ""
---

Ha pasado mucho tiempo desde que, en febrero de 2019, expliqué [cómo instalar MDT 8456 con ADK for Windows 10, version 1809](instalacion-de-mdt-8456).

Durante estos años, dado que [MDT únicamente soporta **Windows 10 1809**](https://learn.microsoft.com/en-us/mem/configmgr/mdt/release-notes#supported-platforms){:target=_blank}, ha sido necesario aplicar diferentes _workarounds_ para que funcione con Windows 11 y los [últimos Windows ADK](https://learn.microsoft.com/en-us/windows-hardware/get-started/adk-install){:target=_blank} de este sistema operativo.

En este artículo explicaré cómo instalar y usar [**MDT 8456**](https://www.microsoft.com/en-us/download/details.aspx?id=54259){:target=_blank} (Enero de 2019) con el último Windows ADK disponible a día de hoy, [**Windows ADK 24H2 10.1.26100.1**](https://go.microsoft.com/fwlink/?linkid=2271337){:target=_blank} (Mayo de 2024).

# Instalación de MDT

## Descarga de instaladores

Para realizar esta instalación hay que descargar los siguientes instaladores:

* [Microsoft Deployment Toolkit 8456 x64](https://download.microsoft.com/download/3/3/9/339BE62D-B4B8-4956-B58D-73C4685FC492/MicrosoftDeploymentToolkit_x64.msi){:target=_blank}
* [Windows ADK 24H2 10.1.26100.1 (May 2024)](https://go.microsoft.com/fwlink/?linkid=2271337){:target=_blank}
* [Windows PE add-on for the Windows ADK 10.1.26100.1 (May 2024)](https://go.microsoft.com/fwlink/?linkid=2271338){:target=_blank}

## Instalación de Windows ADK

* Ejecutar **adksetup.exe** y seguir el asistente de instalación
* Instalar en `C:\Program Files (x86)\Windows Kits\10` marcando las opciones para:
    * No enviar datos de uso anónimos acerca de los Windows 10 Kits
    * Aceptar la licencia del producto
* Seleccionar para instalar el siguiente componente imprescindible:
    * **Deployment Tools** (DISM, SIM, OSCDIMG, BCDBoot, etc.)

{: .box-note}    
Opcionalmente, si se quisieran crear [**paquetes de aprovisionamiento**](https://learn.microsoft.com/es-es/windows/configuration/provisioning-packages/provision-pcs-for-initial-deployment){:target=_blank}, habría que instalar también los siguientes componentes:<br>- Imaging and Configuration Designer (ICD)<br>- Configuration Designer<br>- User State Migration Tool (USMT)

## Instalación de WinPE add-on

* Ejecutar **adkwinpesetup.exe** y seguir el asistente de instalación
* Instalar en `C:\Program Files (x86)\Windows Kits\10` marcando las opciones para:
    * No enviar datos de uso anónimos acerca de los Windows 10 Kits
    * Aceptar la licencia del producto
* Seleccionar para instalar el siguiente componente:
    * **Windows Preinstallation Environment** (Windows PE)

## Instalación del _toolkit_

* Ejecutar **MicrosoftDeploymentToolkit_x64.msi** y seguir el asistente de instalación
    * Aceptar los términos de licencia
* Seleccionar para instalar todos los componentes:
    * Microsoft Deployment Toolkit
    * Documents
    * Tools and Templates
* Instalar en `C:\Program Files\Microsoft Deployment Toolkit`
* No participar en el CEIP (Customer Experience Inprovement Program)

# _Patching_ de MDT

Para solucionar algunos [problemas introducidos por los diferentes Windows ADK posteriores a MDT](https://learn.microsoft.com/en-us/mem/configmgr/mdt/known-issues){:target=_blank} hay que **modificar** la instalación de MDT.

## Actualizar Microsoft.BDD.Utility.dll

A partir del **ADK for Windows 10, version 2004** es necesario **actualizar el fichero Microsoft.BDD.Utility.dll** para que MDT pueda desplegar una imagen de Windows 10 (o ahora Windows 11) en ordenadores que estén configurados en modo BIOS en lugar de UEFI.

Para actualizar la instalación de MDT hay que hacer lo siguiente:

* Descargar el fichero [MDT_KB4564442.exe](https://support.microsoft.com/en-us/topic/windows-10-deployments-fail-with-microsoft-deployment-toolkit-on-computers-with-bios-type-firmware-70557b0b-6be3-81d2-556f-b313e29e2cb7){:target=_blank}
* Descomprimirlo en un directorio temporal (p.ej. `C:\ADM\Setups\MDT_KB456442`)
* Cerrar la aplicación "_Deployment Workbench_" si estuviera abierta
* Hacer una copia de seguridad de los 2 ficheros originales (hay una versión x86 y otra x64)
  * `C:\Program Files\Microsoft Deployment Toolkit\Templates\Distribution\Tools\x86\Microsoft.BDD.Utility.dll`
  * `C:\Program Files\Microsoft Deployment Toolkit\Templates\Distribution\Tools\x64\Microsoft.BDD.Utility.dll`
* Copiar los 2 nuevos ficheros encima de los ficheros originales
* Copiar los 2 nuevos ficheros encima de los ficheros originales **de cada uno de los _deployment shares_ que tengamos**
  * `C:\DeploymentShare\Tools\x86\Microsoft.BDD.Utility.dll`
  * `C:\DeploymentShare\Tools\x64\Microsoft.BDD.Utility.dll`
* Ejecutar la aplicación "_Deployment Workbench_"
* **Regenerar completamente** la _boot image_ de cada _deployment share_ usando la opción **Update Deployment Share**

Las versiones originales del fichero `Microsoft.BDD.Utility.dll` y sus nuevas versiones se detallan en la siguiente tabla:

| Fichero | Plataforma | Versión original | Versión KB4564442 |
| --- | --- | --- | --- | --- |
| Microsoft.BDD.Utility.dll | x86 | 6.3.8456.1000 | 6.3.8456.1001 |
| Microsoft.BDD.Utility.dll | x64 | 6.3.8456.1000 | 6.3.8456.1001 |

## Agregar soporte para _scripts_ HTA

{: .box-note}
Con Windows ADK 24H2 10.1.26100.1 (May 2024) ya no es necesario aplicar este _workaround_.

En el **ADK for Windows 11, version 22H2** se ha cambiado el _default legacy script engine_ y las aplicaciones HTA han dejado de funcionar mostrando el mensaje `Script Error - An error has occurred in the script on this page`:

![Fix HTA][3]

El _workaround_ para que estas aplicaciones, basadas en MSHTML, vuelvan a funcionar consiste en añadir la siguiente clave al registro del entorno Windows PE:

```
reg.exe add "HKLM\Software\Microsoft\Internet Explorer\Main" /t REG_DWORD /v JscriptReplacement /d 0 /f
```

El procedimiento para hacerlo es el siguiente:

* Hacer una copia de seguridad del fichero `C:\Program Files\Microsoft Deployment Toolkit\Templates\Unattend_PE_x64.xml`
* Reemplazar el contenido de ese fichero por el siguiente:

```xml
<unattend xmlns="urn:schemas-microsoft-com:unattend">
    <settings pass="windowsPE">
        <component name="Microsoft-Windows-Setup" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State">
            <Display>
                <ColorDepth>32</ColorDepth>
                <HorizontalResolution>1024</HorizontalResolution>
                <RefreshRate>60</RefreshRate>
                <VerticalResolution>768</VerticalResolution>
            </Display>
            <RunSynchronous>
                <RunSynchronousCommand wcm:action="add">
                    <Description>Fix HTA scripts error Windows 11 ADK 22H2</Description>
                    <Order>1</Order>
                    <Path>reg.exe add "HKLM\Software\Microsoft\Internet Explorer\Main" /t REG_DWORD /v JscriptReplacement /d 0 /f</Path>
                </RunSynchronousCommand>
                <RunSynchronousCommand wcm:action="add">
                    <Description>Lite Touch PE</Description>
                    <Order>2</Order>
                    <Path>wscript.exe X:\Deploy\Scripts\LiteTouch.wsf</Path>
                </RunSynchronousCommand>
            </RunSynchronous>
        </component>
    </settings>
</unattend>
```

* **Regenerar completamente** la _boot image_ de cada _deployment share_ usando la opción **Update Deployment Share**

# _Patching_ de ADK

Para solucionar algunos [problemas introducidos por los diferentes Windows ADK posteriores a MDT](https://learn.microsoft.com/en-us/mem/configmgr/mdt/known-issues){:target=_blank} hay que **modificar** la instalación de Windows ADK.

## Agregar Windows PE x86

A partir del **ADK for Windows 11, version 22H2** no se incluyen los componentes de 32-bits de Windows PE (`WinPE_OCs`).

Esto hace que, al acceder a la pestaña `Windows PE` de un _Deployment Share_ se produzca un error `MMC has detected an error in a snap-in and will unload it` debido a una excepción `System.IO.DirectoryNotFoundException`:

![MMC Crash][1]

El mensaje de error `Could not find a part of the path 'C:\Program Files (x86)\Windows Kits\10\Assessment and Deployment Kit\Windows Preinstallation Environment\x86\WinPE_OCs'` es debido a que **no existe el directorio con los componentes Windows PE de 32-bits**.

El _workaround_ consiste en **crear el directorio `C:\Program Files (x86)\Windows Kits\10\Assessment and Deployment Kit\Windows Preinstallation Environment\x86\WinPE_OCs` vacío**.

{: .box-note}
Si no se necesitan crear _boot images_ de 32-bits no es necesario hacer nada más ya que MDT únicamente comprueba la existencia de ese directorio al acceder a la pestaña `Windows PE`.

Si se quisiera crear una _boot image_ de 32-bits (**aunque ya no estén soportadas por Microsoft**) es necesario descargar la última versión del ADK que incluía los componentes Windows PE de 32-bits:

* [ADK for Windows 11 10.1.22000.1](https://go.microsoft.com/fwlink/?linkid=2165884){:target=_blank}
* [Windows PE add-on for the ADK for Windows 11 10.1.22000.1](https://go.microsoft.com/fwlink/?linkid=2166133){:target=_blank}

A continuación, se pueden instalar _en una máquina temporal_ y copiar el directorio que contiene los componentes Windows PE de 32-bits usando un _script_ similar al siguiente:

```
mkdir D:\ADK_WinPE_x86\x86
robocopy "C:\Program Files (x86)\Windows Kits\10\Assessment and Deployment Kit\Windows Preinstallation Environment\x86" D:\ADK_WinPE_x86\x86 /mir
```

Finalmente, se copia este directorio a la máquina donde se haya instalado Windows ADK 24H2 y se agregan al mismo usando un _script_ similar al siguiente:

```
mkdir "C:\Program Files (x86)\Windows Kits\10\Assessment and Deployment Kit\Windows Preinstallation Environment\x86"
robocopy D:\ADK_WinPE_x86\x86 "C:\Program Files (x86)\Windows Kits\10\Assessment and Deployment Kit\Windows Preinstallation Environment\x86"  /mir
```

Si este procedimiento se ha realizado correctamente, se podrá acceder, sin que se cierre la MMC, a la pestaña `Windows PE` de las propiedades de un _Deployment Share_.

## Agregar soporte para VBScript

{: .box-note}
Con Windows ADK 24H2 10.1.26100.1 (May 2024) ya no es necesario aplicar este _workaround_.

**Windows ADK 10.1.25398.1 (September 2023)** no incluye soporte para VBScript por lo que cualquier _script_ de MDT, como por ejemplo `LiteTouch.wsf` no se puede ejecutar correctamente:

![Missing VBScript][2]

El _workaround_ consiste en obtener los **VBScript FOD** (_Feature On Demand_) desde un **Windows 11 23H2 (Build 25398)** para añadirlos a los componentes `Windows PE` de Windows ADK 24H2.

Mi script [`GetVBScriptFOD.ps1`](https://gist.github.com/manelrodero/2ca6e8d0ba9e770973e3ed9ed89f5462){:target=_blank} permite obtener fácilmente los ficheros que contienen la característica bajo demanda **VBScript**:

```
Microsoft-Windows-VBSCRIPT-FoD-Package~31bf3856ad364e35~amd64~~.cab
Microsoft-Windows-VBSCRIPT-FoD-Package~31bf3856ad364e35~wow64~~.cab
Microsoft-Windows-VBSCRIPT-FoD-Package~31bf3856ad364e35~amd64~en-us~.cab
Microsoft-Windows-VBSCRIPT-FoD-Package~31bf3856ad364e35~wow64~en-us~.cab
```

Una vez obtenidos, únicamente es necesario copiarlos en el directorio `WinPE_OCs` del ADK correspondiente a cada lenguaje (los dos primeros son neutros y los dos segundos corresponden a `en-us`):

```
C:\Program Files (x86)\Windows Kits\10\Assessment and Deployment Kit\Windows Preinstallation Environment\amd64\WinPE_OCs\
   Microsoft-Windows-VBSCRIPT-FoD-Package~31bf3856ad364e35~amd64~~.cab
   Microsoft-Windows-VBSCRIPT-FoD-Package~31bf3856ad364e35~wow64~~.cab

C:\Program Files (x86)\Windows Kits\10\Assessment and Deployment Kit\Windows Preinstallation Environment\amd64\WinPE_OCs\en-us\
   Microsoft-Windows-VBSCRIPT-FoD-Package~31bf3856ad364e35~amd64~en-us~.cab
   Microsoft-Windows-VBSCRIPT-FoD-Package~31bf3856ad364e35~wow64~en-us~.cab
```

# Configuración y uso de MDT

{: .box-note}
En este artículo no detallaré como configurar y usar MDT ya que el artículo que escribí cuando [instalé MDT 8456 con ADK for Windows 10, version 1809](instalacion-de-mdt-8456) sigue siendo válido a día de hoy.

Los pasos principales para configurar y usar Microsoft Deployment Toolkit son los siguientes:

* Crear un **_Deployment Share_** y protegerlo a nivel de ACLs y compartición SMB
* Importar los **sistemas operativos** que se quieran desplegar
* Agregar **paquetes** con actualizaciones o lenguajes (no es necesario si se utiliza [**OSDBuilder**](https://github.com/manelrodero/OSDBuilder){:target=_blank})
* Agregar **aplicaciones** que se instalarán después de desplegar el SO (por ejemplo, los _runtime de Visual C++_)
* Crear una **secuencia de tareas** (_task sequence_) con los pasos que se quieran realizar durante el despliegue
* Modificar los ficheros `Bootstrap.ini` y `CustomSettings.ini` para configurar las **reglas** del despliegue (¿hay que capturar o realizar un _sysprep_ al final del proceso?, ¿hay que unir el equipo a un dominio?, etc.)
* Configurar las opciones de **Windows PE** para definir qué arquitectura tendrá la **imagen de arranque** (_boot image_), qué **controladores** se añadirán, etc.
* Actualizar el _Deployment Share_ para **regenerar** la _boot image_
* Usar esta _boot image_ en una máquina virtual o física para **desplegar** el sistema operativo

# Referencias

* [Toolkit Reference for the MDT](https://docs.microsoft.com/en-us/sccm/mdt/toolkit-reference){:target=_blank}
* [Task sequence variables](https://learn.microsoft.com/en-us/mem/configmgr/osd/understand/task-sequence-variables){:target=_blank}
* [Fixing VBScript Support in Windows ADK SEP 2023 Update (Build 25398)](https://www.deploymentresearch.com/fixing-vbscript-support-in-windows-adk-sep-2023-update-build-25398/){:target=_blank}
* [Windows 11 Deployment – Using MDT 8456 with Windows ADK 23H2 (Build 25398)](https://www.deploymentresearch.com/windows-11-deployment-using-mdt-8456-with-windows-adk-23h2-build-25398/){:target=_blank}

### Historial de cambios

* **2024-10-08**: Documento inicial
* **2024-10-09**: Documentar _workarounds_

[1]: /assets/img/blog/2024-10-08_image_1.png "MMC Crash"
[2]: /assets/img/blog/2024-10-08_image_2.png "Missing VBScript"
[3]: /assets/img/blog/2024-10-08_image_3.png "Fix HTA"
