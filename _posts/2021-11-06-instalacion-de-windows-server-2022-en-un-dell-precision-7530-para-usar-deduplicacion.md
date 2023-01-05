---
layout : post
blog-width: true
title: 'Instalación de Windows Server 2022 en un Dell Precision 7530 para usar deduplicación'
date: '2021-11-06 13:54:00'
published: true
tags:
- Windows
author:
  display_name: Manel Rodero
---

La idea de realizar la instalación de [Windows Server 2022](https://docs.microsoft.com/en-us/windows-server/get-started/whats-new-in-windows-server-2022) en mi [portátil Dell Precisión 7530](/about/laptop) surgió de la necesidad de utilizar las opciones de [**deduplicación de datos**](https://docs.microsoft.com/en-us/windows-server/storage/data-deduplication/install-enable) que proporciona este sistema operativo. Estas opciones de deduplicación de datos no están disponibles en Windows 10 de manera oficial.

La deduplicación de datos analiza los ficheros para encontrar patrones que se repiten y, en lugar de guardar todos los patrones repetidos, se guarda únicamente una copia y se indexa el resto. Cuando se trabaja con **máquinas virtuales**, al ser éstas muy parecidas, el ahorro de espacio puede ser muy importante.

{: .box-note}
**Nota**: Esta idea no es nueva, ya que [Niall Brady](https://twitter.com/ncbrady) instaló [Windows Server 2019 en un Lenovo P1](https://www.niallbrady.com/2019/02/09/installing-windows-server-2019-on-a-lenovo-p1-for-data-dedup-my-rough-notes/) por el mismo motivo. Algunos de los "trucos" explicados en su artículo también se han usado en esta instalación.

{: .box-warning}
**Atención**: La instalación de Windows Server 2022 en un Dell Precision 7530 no está soportada por Dell. Habitualmente este tipo de equipos se venden con una licencia de Windows 10 Pro for Workstations. En mi caso, el portátil se compró con un sistema operativo Linux por resultar mucho más económico.

# Instalación del SO

La instalación del sistema operativo se ha realizado utilizando una [OSBuild](https://github.com/manelrodero/OSDBuilder/blob/main/Prod/Tasks/OSBuild%20Windows%20Server%202022%20x64%2021H2%20BLANK.json) de **Windows Server 2022 x64 21H2 20348.288** generada mediante el módulo [OSDBuilder](https://osdbuilder.osdeploy.com/) de [David Segura](https://twitter.com/seguraosd).

Se ha realizado el siguiente particionado [GPT/UEFI](https://github.com/manelrodero/OSDBuilder/blob/main/Z-Extra/Diskpart_UEFI.txt) del disco 0 (un Samsung PM981 MZ-VLB256A) utilizando el comando `diskpart.exe /s Diskpart_UEFI.txt` desde una ventana de comandos (**Shift+F10** desde la pantalla de instalación):

```
Partition ###  Type              Size     Offset
-------------  ----------------  -------  -------
Partition 1    System             260 MB  1024 KB
Partition 2    Reserved           128 MB   261 MB
Partition 3    Primary            237 GB   389 MB
Partition 4    Recovery           984 MB   237 GB
```

Esta OSBuild utiliza un fichero [`AutoUnattend.xml`](https://github.com/manelrodero/OSDBuilder/blob/main/Z-Extra/AutoUnattend-ServerStandard.xml) que configura algunos aspectos del sistema operativo (idioma y teclado, zona horaria, cuenta principal, etc.).

También se copian [algunos scripts](https://github.com/manelrodero/OSDBuilder/tree/main/Share/Content/ExtraFiles/Windows%2010%20Scripts/Scripts) en `C:\Scripts` que pueden ser de ayuda para acabar la configuración del equipo.

{: .box-error}
**Error**: Aunque durante la instalación se ha podido utilizar sin problema el _touchpad_ del portátil, una vez dentro del sistema operativo este componente no funciona. Este problema ya se había reportado con anterioridad en [Windows Server 2016/Latitude 5580](https://www.dell.com/community/Latitude/Why-can-t-Windows-2016-see-the-touchpad-on-my-Laititude-5580/td-p/6105295), [Windows Server 2016/Precision 7520](https://www.dell.com/community/Precision-Mobile-Workstations/Precision-7520-Touchpad-NOT-working-with-Windows-Server-2016/td-p/6076792) o [Windows Server 2019/Latitude 7490](https://www.dell.com/community/Latitude/Latitude-7490-No-Driver-for-Touchpad-in-Windows-Server-2016-2019/td-p/7481239).

# Actualización del SO

Después de instalar el sistema operativo, se ha procedido a comprobar e instalar cualquier actualización disponible en Windows Update (`Settings > Update & Security > Windows Update`):

![Windows Update][1]

# Instalación de controladores

La instalación **automática** de drivers en un equipo Dell se realiza mediante la aplicación [**Dell Command Update**](https://www.dell.com/support/kbdoc/es-es/000177325/dell-command-update) (DCU). Esta aplicación permite actualizar la BIOS, el firmware, los controladores y las aplicaciones desde una interfaz gráfica o mediante la línea de comandos (`dcu-cli.exe`).

Se ha instalado la versión [4.3.0](https://www.dell.com/support/home/en-us/drivers/DriversDetails?driverId=8D5MC), del 29 de julio de 2021, en su versión estándar (no la aplicación universal para Windows 10). Al ejecutarla por primera vez, se configura para que no compruebe actualizaciones de forma automática y así tener un mayor control sobre las mismas.

Antes de utilizar la opción **Advanced Driver Restore for Windows Re-installation** para aplicar el último [_driver pack_](https://www.dell.com/support/kbdoc/en-us/000124115/precision-7530-windows-10-driver-pack) de este portátil, se descarga la versión [A13](https://downloads.dell.com/FOLDER07546243M/1/7530-win10-A13-4PJ79.CAB), del 25 de agosto de 2021, y se configura como librería en `Settings > Advanced Driver Restore`.

![Dell Command Update][2]

Una vez **reiniciado** el equipo para aplicar correctamente los 25 drivers que forman parte de esta librería, se observa que hay 4 dispositivos que  tienen algún tipo de problema:

* Network Controller (`PCI\VEN_8086&DEV_2526&SUBSYS_40108086&REV_29\4&283BFC7A&0&00E6`)
* Unknown device (`ACPI\DELL0831\4&2A3E7082&0`)
* Unknown device (`BTH\MS_BTHPAN\6&2FA275E4&0&2`)
* Microsoft Usbccid Smartcard Reader (UMDF2) (`USB\VID_0A5C&PID_5833&MI_01\6&1FF64837&0&0001`)

![Dell Command Update][4]

# Adaptador de red Wi-Fi

El dispositivo `PCI\VEN_8086&DEV_2526` corresponde al adaptador de red Wi-Fi. No se han instalado drivers para este dispositivo porque el sistema operativo Windows Server tiene la característica **Wireless LAN Service** desactivada por defecto.

Para habilitar esta característica se puede utilizar la opción `Add roles and features` desde el _Server Manager_ o los siguientes comandos desde PowerShell:

```PowerShell
Install-WindowsFeature Wireless-Networking
Restart-Computer
```

A continuación es necesario activar el servicio **WLAN AutoConfig** desde el Administrador de Servicios o mediante el siguiente comando desde PowerShell:

```PowerShell
Start-Service -Name WlanSvc
```

Finalmente, se puede actualizar el controlador del dispositivo desde el _Device Manager_. Aparecerá una tarjeta **Intel(R) Wireless-AC 9260 160MHz** y se podrán ver todas las redes Wi-Fi disponibles.

# Bluetooth

Otro dispositivo que es necesario reparar es el `BTH\MS_BTHPAN\6&2FA275E4&0&2` correspondiente al **Personal Area Network Service**. Esto se puede hacer actualizando el controlador desde el _Device Manager_:

* Seleccionar la pestaña Driver
* Seleccionar `Update Driver > Browse my computer for drivers > Let me pick from a list of available drivers on my computer > Bluetooth`
* Seleccionar `Microsoft > Personal Area Network Service`
* Aceptar la advertencia sobre la actualización del controlador

# Instalación de más controladores

A continuación se puede volver a utilizar **DCU** para buscar actualizaciones de los controladores que ya están instalados. 

![Dell Command Update][3]

{: .box-warning}
**Atención**: Es muy importante no instalar la [BIOS 1.17.0](https://www.dell.com/support/home/en-us/drivers/DriversDetails?driverId=1RKCW&lwp=rt) ya que tiene un problema en este equipo al [quedarse congelado cuando se graba la configuración](https://twitter.com/manelrodero/status/1456392745564704770).

# I2C HID Device

El dispositivo `ACPI\DELL0831\4&2A3E7082&0` se carga y funciona correctamente en el entorno Windows PE durante la instalación del sistema operativo pero no funciona dentro del sistema operativo.

Se ha utilizado el comando `pnputil.exe` desde Windows PE para exportar el controlador que se inicia correctamente en ese entorno (`hidi2c.sys`):

```
Instance ID:                ACPI\DELL0831\4&2a3e7082&0
Device Description:         I2C HID Device
Class Name:                 HIDClass
Class GUID:                 {745a17a0-74d3-11d0-b6fe-00a0c90f57da}
Manufacturer Name:          Microsoft
Status:                     Started
Driver Name:                hidi2c.inf
```

A continuación se ha forzado la instalación del driver anterior en el sistema operativo mediante el comando `pnputil.exe /add-driver hidi2c.inf` pero el dispositivo siguie teniendo problemas:

```
C:\Drivers>pnputil /enum-devices /problem /deviceids
Microsoft PnP Utility

Instance ID:                ACPI\DELL0831\4&2a3e7082&0
Device Description:         Unknown
Class Name:                 Unknown
Class GUID:                 Unknown
Manufacturer Name:          Unknown
Status:                     Problem
Problem Code:               28 (0x1C) [CM_PROB_FAILED_INSTALL]
Problem Status:             0xC0000490
Hardware IDs:               ACPI\VEN_DELL&DEV_0831
                            ACPI\DELL0831
                            *DELL0831
Compatible IDs:             ACPI\PNP0C50
                            PNP0C50
```

El error [0xC0000490](https://docs.microsoft.com/en-us/windows-hardware/drivers/install/cm-prob-failed-install) o `STATUS_PNP_NO_COMPAT_DRIVERS` quiere decir que los **Hardware ID** especificado en el fichero `*.inf` no son compatibles con el dispositivo o que la **TargetOSVersion** no aplica a la arquitectura y versión del sistema operativo.

Si se descargan el [Dell Touchpad Driver](https://www.dell.com/support/home/en-us/drivers/driversdetails?driverid=2vv2n&oscode=wt64a&productcode=precision-15-7530-laptop) y se extrae su contenido, se comprueba que la única referencia que parece corresponder a este dispositivo (HID\DELL0831&Col03) se encuentra en el fichero `I2CG9Dx64\ApHidFiltrSWD.inf`. La descripción de este fichero indica **AlpsAlpine I2C HID Device Driver for x64 Windows**.

{: .box-warning}
**Atención**: Se podría intentar modificar el fichero `*.inf` y permitir la instalación de drivers no firmados tal como hizo Niall en su artículo [How to enable Thunderbolt on Windows Server 2019](https://www.niallbrady.com/2019/06/26/how-to-enable-thunderbolt-on-windows-server-2019/).

# Smartcard Reader

El dispositivo `USB\VID_0A5C&PID_5833&MI_01\6&1FF64837&0&0001` tampoco funciona correctamente en el sistema operativo generando el error [0xC0000001](https://docs.microsoft.com/en-us/windows-hardware/drivers/install/cm-prob-failed-add):

```
C:\Drivers>pnputil /enum-devices /problem /deviceids
Microsoft PnP Utility

Instance ID:                USB\VID_0A5C&PID_5833&MI_01\6&1ff64837&0&0001
Device Description:         Microsoft Usbccid Smartcard Reader (UMDF2)
Class Name:                 SmartCardReader
Class GUID:                 {50dd5230-ba8a-11d1-bf5d-0000f805f530}
Manufacturer Name:          Microsoft
Status:                     Problem
Problem Code:               31 (0x1F) [CM_PROB_FAILED_ADD]
Problem Status:             0xC0000001
Driver Name:                UsbccidDriver.inf
Hardware IDs:               USB\VID_0A5C&PID_5833&REV_0101&MI_01
                            USB\VID_0A5C&PID_5833&MI_01
Compatible IDs:             USB\Class_0b&SubClass_00&Prot_00
                            USB\Class_0b&SubClass_00
                            USB\Class_0b
```

# Habilitar Audio

En Windows Server 2022 el Audio está desactivado por defecto. Como se quiere poder reproducir sonido en este equipo, es necesario habilitar el servicio **Windows Audio** mediante los siguientes comandos desde PowerShell:

```PowerShell
Set-Service -Name Audiosrv -StartupType Automatic
Start-Service -Name Audiosrv
```

# Configuración de Energía (Opcional)

El sistema operativo Windows Server está optimizado para priorizar los procesos en _background_ (se supone que los usuarios no suelen iniciar sesión en los servidores y no se necesita dar prioridad al escritorio).

Pero como este equipo se utilizará como _HomeLab_, se ha configurado como activo el perfil de energía **High performance** mediante los siguientes comandos desde PowerShell:

```
powercfg.exe /s 8c5e7fda-e8bf-4a96-9a85-a6e23a8c635c
powercfg.exe /setacvalueindex 8c5e7fda-e8bf-4a96-9a85-a6e23a8c635c 4f971e89-eebd-4455-a8de-9e59040e7347 5ca83367-6e45-459f-a27b-476b1d01c936 0
```

# Samsung Magician

Aunque no es imprescindible, la instalación de [Samsung Magician](https://www.samsung.com/semiconductor/minisite/ssd/product/consumer/magician/) permite visualizar el estado de los discos Samsung del equipo (salud, temperatura, etc.):

![Samsung Magician][5]

El almacenamiento del portátil está formado por 3 unidades SSD NVMe M.2:

* Disco 0 (250GB) - Samsung PM981 MZ-VLB256A
* Disco 1 (1TB) - [Samsung 970 EVO Plus](https://www.samsung.com/semiconductor/minisite/ssd/product/consumer/970evoplus/)
* Disco 2 (1TB) - [Samsung 970 EVO Plus](https://www.samsung.com/semiconductor/minisite/ssd/product/consumer/970evoplus/)

# Hyper-V

Para completar la instalación de este _HomeLab_ es necesario habilitar el _rol_ **Hyper-V** desde la opción `Add Roles and Features` del _Server Manager_ o mediante los siguientes comandos desde PowerShell:

```
Install-WindowsFeature -Name Hyper-V -IncludeManagementTools
Restart-Computer
```

# Deduplicación

Para activar la deduplicación es necesario habilitar el _rol_ **Data deduplication** desde la opción `Add Roles and Features` del _Server Manager_ o mediante los siguientes comandos desde PowerShell:

```
Install-WindowsFeature -Name FS-Data-Deduplication
Enable-DedupVolume -Volume V:\ -UsageType HyperV
```

# Configuración de Windows

La configuración final de Windows es una mezcla de los parámetros incluidos en la imagen WIM y la ejecución de diferentes scripts ubicados en `C:\Scripts`:

* Activar Windows
* Opciones del Explorador de Archivos (Offline WIM)
* Deshabilitar AutoPlay (Offline WIM)
* Preferir IPv4 en lugar de IPv6 (AutoUnattend.xml)
* Configurar variable `DIRMCD` (AutoUnattend.xml)
* Deshabilitar Hibernation (AutoUnattend.xml)
* Mostrar dispositivos no presentes en el Administrador de dispositivos (AutoUnattend.xml)
* Añadir `D:\SOFT\Bin` a la variable PATH del sistema
* Habilitar ICMP v4 mediante `C:\Scripts\Enable-ICMPv4.ps1`
* Deshabilitar Netbios mediante `C:\Scripts\Disable-Netbios.ps1`
* Renombrar el equipo mediante `C:\Scripts\Rename-PC.ps1`
* Crear usuarios del equipo mediante `C:\Scripts\Create-Users.ps1` (utiliza el fichero `C:\Scripts\%ComputerName%.users`)
* Accesos directo a Shutdown, Restart y Logoff mediante `C:\Scripts\Copy-Icons.ps1`
* Establecer la imagen de cada usuario desde `C:\Users\Public\Pictures`
* Establecer el wallpaper de la pantalla de bloqueo mediante `C:\Scripts\Config-LockScreenWallpaper.ps1`
* Actualizar ayuda de PowerShell mediante el *cmdlet* `Update-Help -Force`
* Recibir actualizaciones de otros productos Microsoft mediante `C:\Scripts\Config-MicrosoftUpdate.ps1`
* Ajustar las opciones de rendimiento "Efectos visuales" para tener el mejor rendimiento

[1]: /assets/img/blog/2021-11-06_image_1.png "Windows Update"
[2]: /assets/img/blog/2021-11-06_image_2.png "Dell Command Update"
[3]: /assets/img/blog/2021-11-06_image_3.png "Dell Command Update"
[4]: /assets/img/blog/2021-11-06_image_4.png "Device Manager"
[5]: /assets/img/blog/2021-11-06_image_5.png "Samsung Magician"
