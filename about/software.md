---
layout: page
title: About my software
blog-width: true
---

En mis equipos suelo instalar las aplicaciones en diferentes formatos:

* [**Portable**](#portable) (`D:\SOFT\PORTABLE`), para tenerlas disponibles siempre que reinstalo el Sistema Operativo
* [**Run**](#run) (`D:\SOFT\RUN`), son aquellas aplicaciones que realmente no tienen una versión portable pero se pueden ejecutar sin instalar
* [**PortableApps**](#portableapps) (`D:\SOFT\PortableApps`), son las aplicaciones de la plataforma [Portableapps.com](https://portableapps.com/)
* [**Win32**](#win32), son las aplicaciones que se instalan en la unidad `C:\`
* [**UWP**](#uwp), son las aplicaciones instaladas desde la [Microsoft Store](https://www.microsoft.com/es-us/store/apps/)

# <a name="portable">Portable

# <a name="run">Run

# <a name="portableapps">PortableApps

# <a name="win32">Win32

| [7-Zip](#7zip) | [BIG-IP Edge Client](#bigip) | [DNIe](#dnie) | [Docker Desktop](#docker) | [Eduroam](#eduroam) | [Gemalto IDGo 800](#gemalto) |
| [Macrium Reflect Free](#macrium) | [Office Professional Plus 2019](#office) | [Oracle VM VirtualBox](#virtualbox) | [VMware Workstation Pro](#vmware) | | |

## <a name="7zip">7-Zip

| Nombre | [7-Zip](https://www.7-zip.org/) |
| Descripción | 7-Zip is a file archiver with a high compression ratio |
| Empresa | [Igor Pavlov](https://www.7-zip.org/) |
| Versión | 19.00 |
| Release | 21 Febrero 2019 |
| Install | 17 Enero 2021 |

La instalación manual es bastante sencilla:

* Ejecutar el fichero `7z1900-x64.exe`
* Instalar en `C:\Program Files\7-Zip`

Una vez instalado, ejecutar **7-Zip File Manager** y configurarlo de la siguiente manera:

* Tools > Options
* System > Associate 7-Zip with > All extensions/All users

## <a name="bigip">BIG-IP Edge Client

| Nombre | [F5 BIG-IP Edge Client](https://techdocs.f5.com/kb/en-us/bigip-edge-apps.html) |
| Descripción | Cliente de VPN para conectarse a equipos F5 |
| Empresa | [F5](https://www.f5.com/) |
| Versión | 71.2018.1130.2122 |
| Release | |
| Install | 10 Enero 2021 |

Tiene algunas [diferencias con la aplicación F5 Access](https://support.f5.com/csp/article/K23653432) que se instala desde la Microsoft Store.

Una vez instalado, en `Control Panel\Programs\Programs and Features` aparecen dos componentes:

* BIG-IP Edge Client
* BIG-IP Edge Client Components (All Users)

Al ejecutarlo por primera vez, se conecta con el servidor upclink.upc.edu que ya está preconfigurado y se actualiza el cliente a la versión **71.2020.0108.2059**. A partir de aquí únicamente hay que introducir las credenciales para conectarse tal como indica la [documentación](https://techdocs.f5.com/en-us/edge-client-7-1-8/big-ip-access-policy-manager-edge-client-and-application-configuration-7-1-8/big-ip-edge-client-for-windows.html).

## <a name="dnie">DNIe

| Nombre | [DNIe](https://www.dnielectronico.es/PortalDNIe/) |
| Descripción | Documento Nacional de Identidad |
| Empresa | [Dirección General de la Policía](https://www.policia.es/) |
| Versión | [14.1.0](https://www.dnielectronico.es/descargas/CSP_para_Sistemas_Windows/Windows_64_bits/DNIe_v14_1_0(64bits).exe) |
| Release | 13 Noviembre 2019 |
| Install | 17 Enero 2021 |

En el servicio de actualización de Microsoft Windows Update se encuentra disponible la nueva versión del driver [DNIe Minidriver for Smart Card](https://www.catalog.update.microsoft.com/Search.aspx?q=DNIe%20Minidriver%20for%20Smart%20Card):

```
Dirección General de la Policía - SmartCard - 1.0.2.9
Drivers (Other Hardware)
11/13/2019
1.0.2.9
```

Este driver basado en la nueva arquitectura **Smart Card Mini-Driver** (conocida también como "Smart Card Module") de Microsoft, funciona en Internet Explorer y en Google Chrome.

* Descargar el fichero `*.cab` desde Windows Update y extraerlo a un directorio
* Introducir un DNIe en el lector de tarjetas
* Actualizar _Tarjeta inteligente desconocida_ &rarr; _DNIe Minidriver for Smart Card_

Mozilla Firefox como utiliza la arquitectura **PKCS#11** seguirá necesitando la instalación del módulo criptográfico y se debe instalar primeramente el navegador Firefox y luego el módulo criptográfico. Se puede instalar de forma **desatendida** usando el paquete `*.msi` y ejecutando el siguiente comando:

```
msiexec.exe /i "DNIe_v14_1_0(64bits).msi" /quiet
```

Las [autoridades de certificación](https://www.dnielectronico.es/PortalDNIe/PRF1_Cons02.action?pag=REF_076) de la DGP están formadas por:

* [AC Raíz](https://www.dnielectronico.es/PortalDNIe/PRF1_Cons02.action?pag=REF_077)
* [AC Subordinadas](https://www.dnielectronico.es/PortalDNIe/PRF1_Cons02.action?pag=REF_078)

## <a name="docker">Docker Desktop

| Nombre | [Docker Desktop](https://www.docker.com/products/docker-desktop) |
| Descripción | Aplicación para la creación y uso compartido de aplicaciones y microservicios en contenedores |
| Empresa | [Docker, Inc.](https://www.docker.com/) |
| Versión | [3.0.4 (51218)](https://desktop.docker.com/win/stable/Docker%20Desktop%20Installer.exe) |
| Release | |
| Install | 11 Enero 2021 |

Descargar y ejecutar el fichero `Docker Desktop Installer.exe` para instalar Docker Desktop.

* Pantalla 'Configuration'
  * No habilitar "Install required Windows components for WSL 2" (ver Nota)
  * No habilitar "Add shortcut to desktop"
* El instalador descargará y extraerá los ficheros de Docker Desktop
* Cerrar el instalador y salir de la sesión
* Al iniciar de nuevo la sesión se comprueban los requisitos

{: .box-note}
**Nota**: Si no se ha instalado el [kernel de Linux para WSL2](https://aka.ms/wsl2kernel) pedirá que se instale y se reinicie Docker Desktop antes de continuar. Si se han forzado las actualizaciones, [el kernel se instala desde Windows Update](https://twitter.com/manelrodero/status/1346928741843423234).

El instalador creará un grupo `docker-users` en el que añadirá al usuario que ha realizado la instalación. Si es necesario que otros usuarios puedan ejecutar Docker, hay que añadirlos al grupo:

```
Add-LocalGroupMember -Name 'docker-users' -Member <username> -ErrorAction SilentlyContinue
```

A continuación se puede configurar Docker Desktop:

* Settings
  * General
    * Start Docker Desktop when you log in &rarr; Disable
    * Use the WSL2 based engine &rarr; Enable
    * Send usage statistics &rarr; Disable

Una vez que se ha instalado, se puede comprobar que funciona haciendo lo siguiente desde una terminal (CMD, PowerShell, Windows Terminal):

```
# Comprobar la versión
docker version

# Verificar que se pueden descargar y ejecutar imágenes
docker run hello-world
```

Aunque se puede instalar Docker directamente en la distribución WSL2, no sería accesible directamente desde Windows. Por ese motivo es mejor utilizar Docker Deskop, donde se tendrá acceso a Docker desde PowerShell/CMD, acceso al sistema de ficheros, posibilidad de cambiar a contenedores Windows y usarlos desde un IDE.

Referencias:

* [Get started with Docker remote containers on WSL 2](https://docs.microsoft.com/windows/wsl/tutorials/wsl-containers)
* [Docker Desktop WSL 2 backend](https://docs.docker.com/docker-for-windows/wsl/)
* [Using Docker in WSL 2](https://code.visualstudio.com/blogs/2020/03/02/docker-in-wsl2)
* [WSL2 for Dockerized .NET Core application](https://subhankarsarkar.com/wsl2-for-containerised-dot-net-core-development-using-docker/)

## <a name="eduroam">Eduroam

| Nombre | [Eduroam CAT](https://cat.eduroam.org/) |
| Descripción | Instalador de Eduroam configurado con los parámetros de cada organización |
| Empresa | [Eduroam](https://www.eduroam.org/) |
| Versión | [1.0.0.0](https://cat.eduroam.org/user/API.php?action=downloadInstaller&api_version=2&lang=en&device=w10&profile=2742) |
| Release | |
| Install | 11 Enero 2021 |

La instalación [silenciosa](https://wiki.geant.org/display/H2eduroam/A+guide+to+eduroam+CAT+for+IdP+administrators#AguidetoeduroamCATforIdPadministrators-SilentWindowsinstallers) mediante el parámetro `/S` no acaba de serlo del todo porque tiene que instalar una CA que no está reconocida, **Go Daddy Class 2 Certification Authority**, y se queda esperando a que el usuario acepte.

Para localizar estos certificados se puede utilizar `certmgr.msc` (Certificados de usuario), `certlm.msc` (Certificados de máquina) o ejecutar la siguiente [consulta en PowerShell](https://stackoverflow.com/questions/56341798/get-ssl-certificate-issued-to):

```
Get-ChildItem -Path Cert:\ -Recurse | Where-Object { $_.Issuer -like "*Go Daddy*" }
```

Los certificados instalados **a nivel de usuario** son:

* Certificate::CurrentUser\Root (Trusted Root Certification Authority)
  * OU=Go Daddy Class 2 Certification Authority
* Certificate::CurrentUser\CA (Intermediate Certification Authority)
  * CN=Go Daddy Root Certificate Authority - G2
  * CN=Go Daddy Secure Certificate Authority - G2

La instalación manual consiste en ejecutar el fichero `eduroam-W10-U-UPdC.exe` y seguir las instrucciones. Al final del proceso, a parte de los certificados instalados, se habrán creado varios perfiles Wi-Fi que se pueden listar mediante el siguiente comando:

```
netsh.exe wlan show profiles
```

Estos perfiles son los siguientes:

* All User Profile : eduroam© via Passpoint
* All User Profile : eduroam

Se pueden exportar estos perfiles usando los siguientes comandos:

```
netsh.exe wlan export profile name="eduroam" folder="c:\export"
netsh.exe wlan export profile name="eduroam© via Passpoint" folder="c:\export"
```

## <a name="gemalto">Gemalto IDGo 800

| Nombre | Gemalto IDGo 800|
| Descripción | Drivers para tarjetas Gemalto IDPrime MD Smart Card |
| Empresa | [Gemalto](https://www.thalesgroup.com/en/markets/digital-identity-and-security/gemalto-website-has-moved-thales) &rarr; [Thales](https://www.thalesgroup.com/en/markets/digital-identity-and-security) |
| Versión | 1.2.4 |
| Release | 16 Octubre 2015 |
| Install | 17 Enero 2021 |

Estos drivers son los utilizados por las [tarjetas R7 de la UPC](https://serveistic.upc.edu/ca/identitatDigital/documentacio/manuals-certificat-digital/guia-certificat-windows).

De igual manera que el DNIe, existe la versión **Mini-Driver** (Edge, Chrome) y las librerías **PKCS#11** (para Mozilla Firefox). Se instalarán los dos componentes ejecutando los ficheros `IDGo800_Minidriver_64.msi` y `IDGo800_PKCS11_Library.msi`.

También existe la herramienta `IDGo800UserTool_v1.1.27.exe` para leer la tarjeta, cambiar el PIN, etc.

* Introducir una tarjeta UPC R7 en el lector de tarjetas
* (Sin drivers) _Tarjeta inteligente desconocida_
* (Con drivers) _Gemalto IDPrime MD Smart Card_

## Macrium Reflect Free

| Nombre | [Macrium Reflect Free](https://www.macrium.com/reflectfree) |
| Descripción | Programa gratuito para hacer para realizar copias de seguridad, imágenes y clonado de discos |
| Empresa | [Paramount Software UK](https://www.macrium.com/) |
| Versión | [7.3.5365](https://updates.macrium.com/reflect/v7/ReflectDLHF.exe) |
| Release | 26 Noviembre 2020 |
| Install | 10 Enero 2021 |

Mediante `ReflectDLHF.exe` (Macrium Reflect Download Agent) se puede descargar la última versión del programa así como los componentes Windows PE para crear los medios de recuperación.

La instalación se realiza ejecutando el archivo `v7.3.5365_reflect_setup_free_x64.exe` descargado con la herramienta anterior:

* Desmarcar `Log` en la pantalla inicial
* Aceptar el acuerdo de licencia
* Licencia de tipo '**Home**'
* No registrar la instalación
* No instalar ViBoot
* No instalar Desktop Shortcut
* Instalar en `C:\Program Files\Macrium\Reflect`
* Marcar `Launch now` en la pantalla final

Se ejecuta el programa y se configura de la siguiente manera:

* Other Tasks > Edit Defaults...
  * Backup
    * Compression > High
    * Auto Verify Image > Marcar "Verify image or backup file directly after creation"
  * Update
    * Update Properties
      * Desmarcar "Hourly background checks for software updates"
      * Desmarcar "Daily checks for software updates when Macrium Reflect loads"

* Other Tasks > Create **Rescue Media**...
  * Advanced
    * Base WIM > WinRE (incluida con Windows 10)
    * Options > Arquitecture > 64-bit
    * Options > Add BitLocker Support
    * Options > Add WiFi Support
  * Build ISO File &rarr; `Macrium_RescueCD_WinRE10.2004x64_7.3.5365.iso`

## <a name="office">Office Professional Plus 2019

| Nombre | [Office Professional Plus 2019](https://docs.microsoft.com/en-us/deployoffice/office2019/overview) |
| Descripción | Herramientas ofimáticas que incluye el editor de textos Word, la hoja de cálculo Excel, etc. |
| Empresa | [Microsoft](https://www.microsoft.com/) |
| Versión | 16.0.10369.20032 |
| Release | |
| Install | 11 Enero 2021 |

La descarga e instalación de Office Professional Plus 2019 se realiza mediante [**Office Deployment Tool**](https://github.com/manelrodero/OSD/tree/main/ODT).

## <a name="virtualbox">Oracle VM VirtualBox

| Nombre | [Oracle VM VirtualBox](https://www.oracle.com/virtualization/virtualbox/) |
| Descripción | Hipervisor de escritorio que permite ejecutar máquinas virtuales |
| Empresa | [Oracle](https://www.oracle.com/) |
| Versión | [6.1.16](https://download.virtualbox.org/virtualbox/6.1.16/VirtualBox-6.1.16-140961-Win.exe) |
| Release | 16 Octubre 2020 |
| Install | 16 Enero 2021 |

La instalación manual consiste en:

* Ejecutar el fichero `VirtualBox-6.1.16-140961-Win.exe`
* Instalación de todos los componentes de VirtualBox
  * Application
  * USB Support
  * Networking
    * Bridged Networking
    * Host-Only Networking
  * Pyhton 2.x Support
* Instalar en `C:\Program Files\Oracle\VirtualBox`
* Marcar la opción "Create start menu entries"
* Desmarcar la opción "Create a shortcut on the desktop"
* Desmarcar la opción "Create a shortcut in the Quick Launch Bar"
* Marcar la opción "Register file associations"
* Instalar el producto
* Instalar los dispositivos de Oracle (**sin confiar** siempre en el software de Oracle)

Una vez instalado, se ejecuta el programa y se configura de la siguiente manera:

* File > Preferences
* General > Default Machine Folder > `D:\EQUIPS\VMs`
* Update > Deshabilitar la opción "Check for updates"
* Language > English
* Extensions > Instalar `Oracle_VM_VirtualBox_Extension_Pack-6.1.16.vbox-extpack`

## <a name="vmware">VMware Workstation Pro

| Nombre | [VMware Workstation Pro](https://www.vmware.com/es/products/workstation-pro.html) |
| Descripción | Hipervisor de escritorio que permite ejecutar máquinas virtuales, contenedores y clústeres de Kubernetes |
| Empresa | [VMware, Inc.](https://www.vmware.com/) |
| Versión | [16.1.0-17198959](https://download3.vmware.com/software/wkst/file/VMware-workstation-full-16.1.0-17198959.exe) |
| Release | 19 Noviembre 2020 |
| Install | 16 Enero 2021 |

En la sección _Product Downloads_ de [My VMware](http://www.vmware.com/go/downloadworkstation) se puede conocer cual es la última versión del producto, el nombre del fichero: `VMware-workstation-full-x.y.z-build.exe`, la _release date_ y acceder a la [documentación](https://docs.vmware.com/en/VMware-Workstation-Pro/16.1.0/rn/VMware-Workstation-1610-Pro-Release-Notes.html). Para descargar desde esta página hay que estar registrado en VMware.

Ahora bien, se puede descargar **VMware Workstation Pro** desde la página de [evaluación](https://www.vmware.com/products/workstation-pro/workstation-pro-evaluation.html) siguiendo el enlace para [Windows](https://www.vmware.com/go/getworkstation-win).

La URL de descarga se construye concatenando `https://download3.vmware.com/software/wkst/file/` y el nombre del fichero `VMware-workstation-full-x.y.z-build.exe` de cada versión concreta.

{: .box-note}
**Nota**: VMware Workstation Pro necesita [**Windows Hypervisor Platform (WHP)**](https://kb.vmware.com/s/article/76918) para funcionar correctamente con Hyper-V.

La instalación manual consiste en:

* Ejecutar el fichero `VMware-workstation-full-x.y.z-build.exe`
* Aceptar el acuerdo de licencia
* Marcar la opción "Install Windows Hypervisor Platform (WHP) automatically"
* Instalar en `C:\Program Files (x86)\VMware\VMware Workstation`
* Marcar la opción "Enhanced Keyboard Driver (a reboot will be required to use this feature)"
* Desmarcar la opción "Add VMware Workstation console tools into system PATH"
* Desmarcar la opción "Check for product updates on startup"
* Desmarcar la opción "Join the VMware Customer Experience Improvement Program"
* Desmarcar la opción "Desktop shortcut"
* Marcar la opción "Start Menu Programs Folder"
* Instalar el producto
* Introducir la clave de licencia
* Reiniciar el equipo

Una vez instalado, se ejecuta el programa y se configura de la siguiente manera:

* Edit > Preferences
* Workspace > Default location for virtual machines > `D:\EQUIPS\VMs`

También se podría instalar de forma desatendida mediante el siguiente comando:

{: .box-note}
**Nota**: VMware Workstation Pro necesita los _Runtime de Visual C++ 2015-2019_. Si no están instalados, se instalarán y será necesario reiniciar para continuar la instalación.

```
VMware-workstation-full-x.y.z-build.exe /s /v /qb! AUTOSOFTWAREUPDATE=0 DATACOLLECTION=0 DATASTORE_PATH=<Path> DESKTOP_SHORCUT=0 EULAS_AGREED=1 KEEP_LICENSE=0 ADDLOCAL=ALL REMOVE="" REBOOT=ReallySuppress SERIALNUMBER="<Serial>"`
```

A continuación se puede configurar la red mediante el siguiente script:

```
SET PROG=%ProgramFiles%
IF "%PROCESSOR_ARCHITECTURE%"=="AMD64" SET PROG=%ProgramFiles(x86)%

START /WAIT "" "%PROG%\VMware\VMware Workstation\vnetlib64.exe" -- stop nat
START /WAIT "" "%PROG%\VMware\VMware Workstation\vnetlib64.exe" -- stop dhcp

START /WAIT "" "%PROG%\VMware\VMware Workstation\vnetlib64.exe" -- set vnet vmnet1 addr 192.168.60.0
START /WAIT "" "%PROG%\VMware\VMware Workstation\vnetlib64.exe" -- set vnet vmnet1 mask 255.255.255.0
START /WAIT "" "%PROG%\VMware\VMware Workstation\vnetlib64.exe" -- set adapter vmnet1 addr 192.168.60.1
START /WAIT "" "%PROG%\VMware\VMware Workstation\vnetlib64.exe" -- remove dhcp vmnet1
START /WAIT "" "%PROG%\VMware\VMware Workstation\vnetlib64.exe" -- remove nat vmnet1
START /WAIT "" "%PROG%\VMware\VMware Workstation\vnetlib64.exe" -- add dhcp vmnet1
START /WAIT "" "%PROG%\VMware\VMware Workstation\vnetlib64.exe" -- update dhcp vmnet1
START /WAIT "" "%PROG%\VMware\VMware Workstation\vnetlib64.exe" -- update nat vmnet1
START /WAIT "" "%PROG%\VMware\VMware Workstation\vnetlib64.exe" -- update adapter vmnet1

START /WAIT "" "%PROG%\VMware\VMware Workstation\vnetlib64.exe" -- set vnet vmnet8 addr 192.168.20.0
START /WAIT "" "%PROG%\VMware\VMware Workstation\vnetlib64.exe" -- set vnet vmnet8 mask 255.255.255.0
START /WAIT "" "%PROG%\VMware\VMware Workstation\vnetlib64.exe" -- set adapter vmnet8 addr 192.168.20.1
START /WAIT "" "%PROG%\VMware\VMware Workstation\vnetlib64.exe" -- set nat vmnet8 internalipaddr 192.168.20.2
START /WAIT "" "%PROG%\VMware\VMware Workstation\vnetlib64.exe" -- add dhcp vmnet8
START /WAIT "" "%PROG%\VMware\VMware Workstation\vnetlib64.exe" -- add nat vmnet8
START /WAIT "" "%PROG%\VMware\VMware Workstation\vnetlib64.exe" -- update dhcp vmnet8
START /WAIT "" "%PROG%\VMware\VMware Workstation\vnetlib64.exe" -- update nat vmnet8
START /WAIT "" "%PROG%\VMware\VMware Workstation\vnetlib64.exe" -- update adapter vmnet8

START /WAIT "" "%PROG%\VMware\VMware Workstation\vnetlib64.exe" -- start nat
START /WAIT "" "%PROG%\VMware\VMware Workstation\vnetlib64.exe" -- start dhcp
```

# <a name="uwp">UWP

| [F5 Access](#f5) | [Ubuntu WSL](#ubuntu) |

## <a name="f5">F5 Access

![Foto: Microsoft.com](https://store-images.s-microsoft.com/image/apps.6267.9007199266584715.3f40adb8-6e4f-433c-9d69-79e14f8208e0.d7fb5707-5965-4974-94e3-8c2494b87ea6?mode=scale&q=90&h=48&w=48&background=%230078D7)

| Nombre | [F5 Access](https://www.microsoft.com/store/productId/9WZDNCRDSFN0) |
| Descripción | Cliente de VPN para conectarse a equipos F5 |
| Empresa | [F5](https://www.f5.com/) |
| Versión | 1.3.0.0 |
| Release | |
| Install | |

Tiene algunas [diferencias con la aplicación BIG-IP Edge Client](https://support.f5.com/csp/article/K23653432) que se descarga desde [la página web de Serveis TIC](https://serveistic.upc.edu/ca/upclink/documentacio/upclink-windows-client).

Una vez instalado, se puede configurar una conexión VPN desde `Settings > Network & Ethernet > VPN` con los siguientes parámetros:

* VPN Provider: F5 Access
* Connection name: UPClink
* Server name or address: upclink.upc.edu
* Remember my sign-in info

## <a name="ubuntu">Ubuntu WSL

![Foto: Microsoft.com](https://store-images.s-microsoft.com/image/apps.63954.14012035919677084.98492fa2-b361-426d-8534-028ce70fe7cc.89345076-a024-4d5f-9406-c76958d41636?mode=scale&q=90&h=48&w=48&background=%23E95420)

| Nombre | [Ubuntu](https://www.microsoft.com/store/productId/9N6SVWS3RX71) |
| Descripción | Distribución de Linux basada en Debian |
| Empresa | [Canonical Group Limited](https://ubuntu.com/) |
| Versión | 20.04 LTS |
| Release | |
| Install | 17 Enero 2021 |

Si se ha instalado [Windows Subsystem for Linux (WSL)](https://docs.microsoft.com/en-us/windows/wsl/) se puede instalar una distribución de Linux como Ubuntu 20.04 desde la Microsoft Store.

Los pasos fundamentales para instalar WSL son:

* Habilitar la característica `Microsoft-Windows-Subsystem-Linux`
* Disponer de Windows 10 1903 o superior
* Habilitar la característica `VirtualMachinePlatform`
* Instalar la actualización del [kernel para WSL2](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)
* Hacer que WSL2 sea la distribución por defecto

```
wsl.exe --set-default-version 2
```
<p></p>

* Instalar una distribución desde la Microsoft Store
* (Opcional) Instalar [Windows Terminal](https://docs.microsoft.com/en-us/windows/terminal/get-started)

También es posible instalar las distribuciones de forma [manual](https://docs.microsoft.com/en-us/windows/wsl/install-on-server) descargando los ficheros `*.appx` para instalarlos mediante `Add-AppxPackage`:

```
curl.exe -L -o wsl-ubuntu-2004.appx https://aka.ms/wslubuntu2004
```

{: .box-note}
**Nota**: Los ficheros se podrían descargar en formato ZIP y usar `Expand-Archive` para descomprimirlos. Después se ejecutaría el fichero `*.exe` de la distribución tal como expliqué [durante la migración de este blog a Jekyll](https://www.manelrodero.com/blog/migracion-de-wordpress-a-jekyll-y-github-pages#instalaci%C3%B3n-de-ubuntu-1804).

El siguiente [artículo de Puget Systems](https://www.pugetsystems.com/labs/hpc/Note-How-To-Copy-and-Rename-a-Microsoft-WSL-Linux-Distribution-1811/) explica como copiar y renombrar una distribución WSL.
