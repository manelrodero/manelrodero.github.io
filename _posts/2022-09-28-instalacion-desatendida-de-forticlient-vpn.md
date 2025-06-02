---
layout : post
blog-width: true
title: 'Instalación desatendida de FortiClient VPN'
date: '2022-09-28 21:53:46'
last-updated: '2025-06-02 11:41:50'
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

{: .box-note}
Este fichero descargado es realmente un _stub_ que se conecta a los servidores de Fortinet para descargar el verdadero instalador `FortiClientVPN.exe`.

* Ejecutar el instalador `FortiClientVPNInstaller.exe` desde el dirctorio temporal

En el fichero `FCTInstall.log` de la carpeta `%TEMP%` se registran las direcciones IP de los servidores de Fortinet desde donde se descarga la última imagen del instalador, la versión del mismo y la ubicación del ejecutable `FortiClientVPN.exe`.

```plaintext
Mon Jun  2 11:47:20 2025 - Server list:
Mon Jun  2 11:47:20 2025 -      209.40.106.66 TZ7
Mon Jun  2 11:47:20 2025 -      173.243.138.76 TZ9
Mon Jun  2 11:47:20 2025 - begin download.
Mon Jun  2 11:47:20 2025 - downloading server list.
Mon Jun  2 11:47:20 2025 - downloading image table, server 209.40.106.66
Mon Jun  2 11:47:21 2025 - Highest available image found: 07004000FIMG03028-00004.00003.
Mon Jun  2 11:47:21 2025 - This image is version: 7.4.3.
Mon Jun  2 11:47:21 2025 - downloading image 07004000FIMG03028-00004.00003
Mon Jun  2 11:47:21 2025 - downloading image from server: 209.40.106.66
Mon Jun  2 11:48:39 2025 - unpacking downloaded image.
Mon Jun  2 11:48:41 2025 - end download.
Mon Jun  2 11:48:43 2025 - Installer filename is: FortiClientVPN.exe
Mon Jun  2 11:48:43 2025 - C:\Users\WDAGUtilityAccount\AppData\Local\Temp\FortiClientVPN.exe
```

Cuando aparezca la pantalla del asistente _Welcome to the FortiClient VPN Setup Wizard_, sin cerrar la ventana ni avanzar en la instalación:

* Acceder a la carpeta `%TEMP%`
* Copiar el fichero `FortiClientVPN.exe` al directorio temporal
* Acceder a la carpeta `C:\ProgramData\Applications\Cache\{15C7B361-A0B2-4E79-93E0-868B5000BA3F}\7.4.3.1790`
* Copiar el fichero `FortiClient.msi` al directorio temporal

{: .box-note}
El directorio `{15C7B361-A0B2-4E79-93E0-868B5000BA3F}\7.4.3.1790` indicado anteriormente se construye mediante un GUID aleatorio y la versión de FortiClient que se está instalando. El siguiente _script_ permite hacer la copia sin conocer la versión.

```powershell
Copy-Item -Path "$env:TEMP\FortiClientVPN.exe" -Destination "C:\FortiClientSetup"
Get-ChildItem -Path "C:\ProgramData\Applications\Cache" -Recurse -Filter "FortiClientVPN.msi" | Copy-Item -Destination "C:\FortiClientSetup"
```

A continuación, se procede a cancelar el asistente de instalación y a salir del instalador.

El contenido del directorio temporal será similar al siguiente:

```
02/06/2025  11:48       212.601.984 FortiClientVPN.exe
02/06/2025  11:48       197.103.616 FortiClientVPN.msi
02/06/2025  11:43         4.511.360 FortiClientVPNInstaller.exe
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
    * Connection Name: `VPN Empresa`
    * Description: `VPN Empresa`
    * Remote Gateway: `<FQDN>` del servidor (p.ej. `vpn.empresa.com`)
    * Enable Single Sign On (SSO) for VPN Tunnel: `Habilitar`
  * Guardar la conexión
* Cerrar el programa
* Exportar el registro de máquina: `reg.exe export HKLM\SOFTWARE\Fortinet hklm2.reg`
* Exportar el registro de usuario: `reg.exe export HKCU\SOFTWARE\Fortinet hkcu2.reg`
* Comparar cada pareja de ficheros (por ejemplo mediante [WinMerge](https://winmerge.org/)):
  * `WinMergeU.exe hklm1.reg hklm2.reg`
  * `WinMergeU.exe hkcu1.reg hkcu2.reg`

Al comparar estos ficheros se observa que se ha añadido la siguiente información en el registro de máquina `HKLM`:

```
[HKEY_LOCAL_MACHINE\SOFTWARE\Fortinet\FortiClient\Sslvpn]

[HKEY_LOCAL_MACHINE\SOFTWARE\Fortinet\FortiClient\Sslvpn\Tunnels]

[HKEY_LOCAL_MACHINE\SOFTWARE\Fortinet\FortiClient\Sslvpn\Tunnels\VPN Empresa]
"Description"="VPN Empresa"
"Server"="vpn.empresa.com:443"
"DATA1"="EncLM 71ba4049 ... (un total de 588 carácteres hexadecimales) ... 9829613b"
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
[HKEY_CURRENT_USER\SOFTWARE\Fortinet\FortiClient\FA_UI]

[HKEY_CURRENT_USER\SOFTWARE\Fortinet\FortiClient\FA_UI\VPN-7.4.3.1790]
"installed"=dword:67dab596

[HKEY_CURRENT_USER\SOFTWARE\Fortinet\FortiClient\Sslvpn]

[HKEY_CURRENT_USER\SOFTWARE\Fortinet\FortiClient\Sslvpn\Tunnels]

[HKEY_CURRENT_USER\SOFTWARE\Fortinet\FortiClient\Sslvpn\Tunnels\VPN Empresa]
"promptusername"=dword:00000000
"promptcertificate"=dword:00000000
"DATA3"=""
```

Después de hacer algunas pruebas, y observar que las claves `DATA1` y `DATA3` no son necesarias porque FortiClient las regenera, se generan los siguientes ficheros con el contenido mínimo indispensable para que un usuario tenga el perfil VPN ya configurado y pueda conectarse sin problemas:

* Fichero `FortiClient-VPNEmpresa-HKLM.reg`

```
Windows Registry Editor Version 5.00

; (Opcional) Deshabilitar las actualizaciones automáticas si los usuarios no son administradores
[HKEY_LOCAL_MACHINE\SOFTWARE\Fortinet\FortiClient\FA_UPDATE]
"enabled"=dword:00000000

; No recordar el usuario en la Web View del SSO para la versión 7.4.0+
[HKEY_LOCAL_MACHINE\SOFTWARE\Fortinet\FortiClient\FA_VPN]
"after_logon_saml_auth"=dword:00000001

; Borrar conexión VPN Empresa existente
[-HKEY_LOCAL_MACHINE\SOFTWARE\Fortinet\FortiClient\Sslvpn\Tunnels\VPN Empresa]

; Crear conexión VPN Empresa
[HKEY_LOCAL_MACHINE\SOFTWARE\Fortinet\FortiClient\Sslvpn\Tunnels\VPN Empresa]
"Description"="VPN Empresa"
"Server"="vpn.empresa.com:443"
"promptusername"=dword:00000000
"promptcertificate"=dword:00000000
"ServerCert"="1"
"sso_enabled"=dword:00000001
"use_external_browser"=dword:00000000
"azure_auto_login"=dword:00000000
```

* Fichero `FortiClient-VPNEmpresa-HKCU.reg`

```
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\SOFTWARE\Fortinet\FortiClient]
"installed"=dword:00000001

; Aceptar el Disclaimer sobre la versión gratuita (el valor hexadecimal depende de la versión)
[HKEY_CURRENT_USER\SOFTWARE\Fortinet\FortiClient\FA_UI\VPN-7.4.3.1790]
"installed"=dword:67dab596

; Borrar conexión VPN Empresa existente
[-HKEY_CURRENT_USER\software\fortinet\FortiClient\Sslvpn\Tunnels\VPN Empresa]

; Crear conexión VPN Empresa
[HKEY_CURRENT_USER\software\fortinet\FortiClient\Sslvpn\Tunnels\VPN Empresa]
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
* Seleccionar la opción _Response Transform_ y seleccionar el fichero `C:\FortiClientSetup\FortiClientVPN.msi`
  * Aparece el asistente _Welcome to the FortiClient VPN Setup Wizard_
    * Marcar la casilla para aceptar el _License Agreement_
    * Pulsar sobre el botón `Next`
    * Pulsar sobre el botón `Install`
    * Pulsar sobre el botón `Finish`
  * Se crea el fichero de transformación `FortiClientVPN.mst`
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
    * Comprobar que se han añadido las claves en `HKEY_LOCAL_MACHINE\Software\Fortinet\FortiClient\Sslvpn\Tunnels\VPN Empresa`
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

```powershell
reg.exe add "HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\Installer" /v "Logging" /t REG_SZ /d "voicewarmupx" /f
```

# Reto _Hacking_

Me gustaría averiguar cómo se calcula el valor hexadecimal de la clave `installed` bajo `HKEY_CURRENT_USER\SOFTWARE\Fortinet\FortiClient\FA_UI\VPN-7.4.3.1790` que sirve para indicar que el usuario ha leído el _disclaimer_.

Parecía que podría estar relacionado con la fecha de la versión o similar, pero no estoy seguro. Si alguien quiere investigar un poco más, aquí le dejo mi primera aproximación para obtener datos:

```powershell
$datosFortiClient = @(
    @{ Version = "7.0.7.0345"; Hexadecimal = "630f39a6" },
    @{ Version = "7.2.2.0864"; Hexadecimal = "651d095a" },
    @{ Version = "7.4.0.1658"; Hexadecimal = "6659c336" },
    @{ Version = "7.4.3.1790"; Hexadecimal = "67dab596" }
)

$baseDate = Get-Date "1970-01-01 00:00:00"

foreach ($dato in $datosFortiClient) {
    $decimal = [convert]::ToInt64($dato.Hexadecimal, 16)
    $fecha = $baseDate.AddSeconds($decimal)

    Write-Output "Versión: $($dato.Version) | Hexadecimal: $($dato.Hexadecimal) | Decimal: $decimal | Fecha estimada: $fecha"
}
```

### Historial de cambios

* **2022-09-28**: Documento inicial (FortiClient 7.0.7.0345)
* **2024-10-07**: Revisión del documento (FortiClient 7.4.0.1658)
* **2025-06-02**: Revisión del documento (FortiClient 7.4.3.1790)