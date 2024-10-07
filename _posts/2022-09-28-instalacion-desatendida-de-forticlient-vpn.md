---
layout : post
blog-width: true
title: 'Instalación desatendida de FortiClient VPN'
date: '2022-09-28 21:53:46'
last-updated: '2024-10-07 08:48:50'
published: true
tags:
- Software
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2024-10-07_cover.png"
thumbnail-img: ""
---

En este artículo se explica como realizar la instalación desatendida de [FortiClient VPN](https://docs.fortinet.com/product/forticlient/7.0) junto a un perfil VPN para que el usuario final ya lo tenga todo configurado.

## Descarga del instalador

Para realizar este tipo de instalación es necesario obtener el **instalador completo** (ya sea en formato `*.exe` o, mucho mejor aún, en formato `*.msi`):

* Navegar a la [página de descargas de Fortinet](https://www.fortinet.com/support/product-downloads#vpn)
* Buscar la sección *FortiClient* &rarr; *FortiClient VPN only*
* Descargar el [instalador para Windows](https://links.fortinet.com/forticlient/win/vpnagent)
* Guardarlo en un directorio temporal (p.ej. `C:\FortiClientSetup`)

Este fichero descargado es realmente un _stub_ que se conecta a los servidores de Fortinet para descargar la imagen del verdadero instalador:

* Ejecutar el instalador `FortiClientVPNOnlineInstaller.exe` desde el dirctorio temporal

{: .box-note}
En el fichero `FCTInstall.log` de la carpeta `%TEMP%` se registran las direcciones IP de los servidores de Fortinet desde donde se descarga la última imagen del instalador, la versión del mismo y la ubicación del ejecutable `FortiClientVPN.exe`.

* Esperar a que aparezca la pantalla del asistente *Welcome to the FortiClient VPN Setup Wizard*
* Sin cerrar esta ventana ni avanzar en la instalación:
  * Acceder a la carpeta `%TEMP%`
  * Copiar el fichero `FortiClientVPN.exe` al directorio temporal
  * Acceder a la carpeta `C:\ProgramData\Applications\Cache\{0DC51760-4FB7-41F3-8967-D3DEC9D320EB}\7.4.0.1658`
  * Copiar el fichero `FortiClient.msi` al directorio temporal

{: .box-note}
El directorio `{0DC51760-4FB7-41F3-8967-D3DEC9D320EB}\7.4.0.1658` se construye mediante un GUID aleatorio y la versión de FortiClient que se está instalando.

* Copiar estos dos ficheros al directorio temporal
* Cerrar el asistente de instalación

El contenido del directorio temporal será similar al siguiente:

```
10/07/2024  08:51 AM       163,995,648 FortiClient.msi
10/07/2024  08:51 AM       176,739,392 FortiClientVPN.exe
10/07/2024  08:49 AM         2,794,560 FortiClientVPNOnlineInstaller.exe
```

## Instalación mediante EXE

A continuación se podría crear un script que instalase el cliente de forma desatendida mediante alguno de los siguientes comandos:

```
# Instalación completamente silenciosa, sin interacción del usuario
FortiClientVPN.exe /quiet /norestart /log "C:\Logs\FortiClient.log"

# Instalación desatendida, con una barra de progreso
FortiClientVPN.exe /passive /norestart /log "C:\Logs\FortiClient.log"
```

Ahora bien, este método requeriría realizar algunas acciones a posteriori para **personalizar la instalación**:

* Borrar el acceso directo que se crea en el escritorio común (`CommonDesktop` = `C:\Users\Public\Desktop`)
* Añadir el perfil VPN para que el usuario final ya lo tenga todo configurado

## Perfil VPN

Para crear el perfil VPN durante la instalación desatendida hay que saber que, en el caso de FortiClient VPN, esta configuración se guarda en el **registro** de Windows.

Por tanto, hay que averiguar qué cambios se producen en el registro cuando se crea una configuración de forma manual y guardarlos en un fichero `*.reg` que se importará durante la instalación desatendida.

Un método bastante sencillo para conocer los cambios es el siguiente:

* Exportar el registro de máquina: `reg.exe export HKLM\SOFTWARE\Fortinet hklm1.reg`
* Exportar el registro de usuario: `reg.exe export HKCU\SOFTWARE\Fortinet hkcu1.reg`
* Ejecutar el programa y realizar los cambios que se consideren oportunos, en este caso:
  * Aceptar el _disclaimer_ acerca de utilizar una versión gratuita y sin soporte de FortiClient VPN
  * Configurar la conexión VPN que se necesite, por ejemplo:
    * VPN: `SSL-VPN`
    * Connection Name: `VPN Custom`
    * Description: `VPN Custom`
    * Remote Gateway: `<FQDN>` del servidor (p.ej. `vpn.contoso.com`)
    * Enable Single Sign On (SSO) for VPN Tunnel: `Habilitar`
  * Grabar la conexión
* Cerrar el programa
* Exportar el registro de máquina: `reg.exe export HKLM\SOFTWARE\Fortinet hklm2.reg`
* Exportar el registro de usuario: `reg.exe export HKCU\SOFTWARE\Fortinet hkcu2.reg`
* Comparar cada pareja de ficheros (por ejemplo mediante [WinMerge](https://winmerge.org/)):
  * `WinMergeU.exe hklm1.reg hklm2.reg`
  * `WinMergeU.exe hkcu1.reg hkcu2.reg`

Al comparar estos ficheros se observa que se ha añadido la siguiente información en el registro de máquina `HKLM`:

```
[HKEY_LOCAL_MACHINE\SOFTWARE\Fortinet\FortiClient\Sslvpn\Tunnels\VPN Custom]
"Description"="VPN Custom"
"Server"="vpn.contoso.com:443"
"DATA1"="EncLM 4a019620d4 ... (un total de 588 carácteres hexadecimales) ... 1bd18b3803"
"promptusername"=dword:00000000
"promptcertificate"=dword:00000000
"DATA3"=""
"ServerCert"="1"
"dual_stack"=dword:00000000
"sso_enabled"=dword:00000001
"use_external_browser"=dword:00000000
"azure_auto_login"=dword:00000000
```

Y la siguiente información en el registro de usuario `HKCU`:

```
[HKEY_CURRENT_USER\SOFTWARE\Fortinet]

[HKEY_CURRENT_USER\SOFTWARE\Fortinet\FortiClient]
"installed"=dword:00000001

[HKEY_CURRENT_USER\SOFTWARE\Fortinet\FortiClient\FA_UI]

[HKEY_CURRENT_USER\SOFTWARE\Fortinet\FortiClient\FA_UI\VPN-7.4.0.1658]
"installed"=dword:6659c336

[HKEY_CURRENT_USER\SOFTWARE\Fortinet\FortiClient\Sslvpn]

[HKEY_CURRENT_USER\SOFTWARE\Fortinet\FortiClient\Sslvpn\Tunnels]

[HKEY_CURRENT_USER\SOFTWARE\Fortinet\FortiClient\Sslvpn\Tunnels\VPN UPClink]
"promptusername"=dword:00000000
"promptcertificate"=dword:00000000
"DATA3"=""
```

Después de hacer algunas pruebas, y observar que las claves `DATA1` y `DATA3` no son necesarias porque FortiClient las regenera, se generan los siguientes ficheros con el contenido mínimo indispensable para que un usuario tenga el perfil VPN ya configurado y pueda conectarse sin problemas:

* Fichero `FortiClient-VPNCustom-HKLM.reg`

```
Windows Registry Editor Version 5.00

; Borrar conexión VPN Custom existente
[-HKEY_LOCAL_MACHINE\SOFTWARE\Fortinet\FortiClient\Sslvpn\Tunnels\VPN Custom]

; Crear conexión VPN Custom
[HKEY_LOCAL_MACHINE\SOFTWARE\Fortinet\FortiClient\Sslvpn\Tunnels\VPN Custom]
"Description"="VPN Custom"
"Server"="vpn.contoso.com:443"
"promptusername"=dword:00000000
"promptcertificate"=dword:00000000
"ServerCert"="1"
"sso_enabled"=dword:00000001
"use_external_browser"=dword:00000000
"azure_auto_login"=dword:00000000
```

* Fichero `FortiClient-VPNCustom-HKCU.reg`

```
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\SOFTWARE\Fortinet\FortiClient]
"installed"=dword:00000001

; Aceptar el Disclaimer sobre la versión gratuita (depende del número de versión)
[HKEY_CURRENT_USER\software\fortinet\FortiClient\FA_UI\VPN-7.4.0.1658]]
"installed"=dword:6659c336

; Borrar conexión VPN Custom existente
[-HKEY_CURRENT_USER\software\fortinet\FortiClient\Sslvpn\Tunnels\VPN Custom]

; Crear conexión VPN Custom
[HKEY_CURRENT_USER\software\fortinet\FortiClient\Sslvpn\Tunnels\VPN Custom]
"promptusername"=dword:00000000
"promptcertificate"=dword:00000000
```

Estos ficheros se deberían instalar para todos los perfiles de usuario **existentes** y **futuros** del ordenador. Esto se puede conseguir usando el script [WriteToHKCUFromSystem.ps1](https://gist.github.com/manelrodero/0e359de5390ced29c566076861d151cc) que se encarga de introducir la información en cada uno de los **SID** en `HKEY_USERS\$sid` y en `C:\Users\Default\NTUSER.DAT` (el registro del usuario por defecto).

## Transformación del MSI

Aunque el método anterior permite instalar FortiClient VPN de forma desatendida y con un perfil personalizado, se puede simplificar la instalación [**transformando**](https://learn.microsoft.com/en-us/windows/win32/msi/about-transforms) el `*.msi`.

Aquellos que lleven mucho tiempo administrando Windows recordarán que el método tradicional para generar un fichero `*.mst` era utilizar el [editor ORCA](https://learn.microsoft.com/en-us/windows/win32/msi/orca-exe) que, aunque aún sigue siendo válido, tiene una interfaz poco amigable.

Por este motivo, se utilizará la versión **Community** de [**Master Packager**](https://www.masterpackager.com/) que es gratuita y permite realizar las operaciones más comunes con los ficheros `*.msi` y `*.mst`.

* Descargar e instalar _Master Packager_ 24.7.9021 o superior en una máquina limpia
* Ejecutar `MasterPackager.exe`
* Seleccionar la opción _Response Transform_ y seleccionar el fichero `C:\FortiClientSetup\FortiClient.msi`
  * Aparece el asistente _Welcome to the FortiClient VPN Setup Wizard_
    * Marcar la casilla para aceptar el _License Agreement_
    * Pulsar sobre el botón `Next`
    * Pulsar sobre el botón `Install`
    * Pulsar sobre el botón `Finish`
  * Se crea el fichero de transformación `FortiClient.mst`
* Seleccionar _View_ &rarr; _Advanced Editor - Overview_
* Ir a la sección **Property** y pulsar sobre el lapiz para editar
  * Buscar la propiedad `DESKTOPSHORTCUT` y cambiar el valor a **0** (evita la creación de accesos directos en el escritorio)
* Ir a la sección **Custom Actions** y pulsar sobre el lapiz para editar
  * Seleccionar _Predefined Actions_
  * Agregar una nueva acción predefinida pulsando sobre `[+]`
    * Seleccionar _Apply HKCU registries to all users_
      * Explorar y seleccionar el fichero `FortiClient-Custom-HKCU.reg`
* Ir a la sección **Registries**
  * Seleccionar la rama `HKEY_LOCAL_MACHINE`
  * Seleccionar la opción _Import Registry_ con el botón derecho
    * Explorar y seleccionar el fichero `FortiClient-Custom-HKLM.reg`
    * Escoger la opción **_Import as x64_**
    * Comprobar que se han añadido las claves en `HKEY_LOCAL_MACHINE\Software\Fortinet\FortiClient\Sslvpn\Tunnels\VPN Custom`
* Grabar el fichero de transformación mediante _File_ &rarr; _Save_
* Salir del programa y renombrar el fichero de transformación a `FortiClientVPN-Custom.mst`

# Instalación mediante MSI

A continuación se podría crear un script que instalase el cliente de forma desatendida mediante alguno de los siguientes comandos:

```
# Instalación completamente silenciosa, sin interacción del usuario
msiexec.exe /i FortiClient.msi TRANSFORMS=FortiClientVPN-Custom.mst REBOOT=ReallySuppress /quiet

# Instalación desatendida, con una barra de progreso
msiexec.exe /i FortiClient.msi TRANSFORMS=FortiClientVPN-Custom.mst REBOOT=ReallySuppress /passive
```

# Depuración de MSI

Se puede activar la depuración de MSI para forzar la creación de un fichero de registro en la carpeta `%TEMP%`:

* Para ver si el instalador EXE inició una instalación de MSI
* Para comprobar si hay errores
* Para comprobar las propiedades modificadas durante la instalación de MSI
* Para omitir agregar el interruptor /l*v a todas las instalaciones de MSI

```
Windows Registry Editor Version 5.00 

[HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\Installer] 
"Logging"="voicewarmupx" 
```

### Historial de cambios

* **2022-09-28**: Documento inicial (FortiClient 7.0.7.0345)
* **2024-10-07**: Revisión del documento (FortiClient 7.4.0.1658)
