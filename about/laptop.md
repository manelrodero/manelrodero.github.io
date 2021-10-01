---
layout: page
title: About my laptop
blog-width: true
---

| [Características](#características) | [Parámetros de la BIOS](#parámetros-de-la-bios) | [Instalación](#instalación) | [Drivers](#drivers) |

Mi portátil es un [**Dell Precision 7530**](https://www.dell.com/en-us/work/shop/port%C3%A1tiles-dell/precision-7530/spd/precision-15-7530-laptop) fabricado el 21 de febrero de 2019 y entregado el 5 de marzo de 2019. Tiene una garantía de 3 años en los siguientes servicios:

| Service | Start date | Expiration date |
| ------- | ---------- | --------------- |
| Complete Care | 22 FEB 2019 | 22 FEB 2022 |
| NBD ProSupport | 22 FEB 2019 | 22 FEB 2022 |
| Battery Exchange Service | 22 FEB 2019 | 22 FEB 2022 |
| Onsite Service After Remote Diagnosis | 22 FEB 2019 | 22 FEB 2022 |

![Foto: dell.com][1]

# Características

* Procesador [Intel Xeon E-2176M](https://ark.intel.com/content/www/es/es/ark/products/134867/intel-xeon-e-2176m-processor-12m-cache-up-to-4-40-ghz.html):
  * Núcleos: 6
  * Hilos: 12
  * Frecuencia básica: 2,70 GHz
  * Frecuencia máxima (Turbo): 4,40 GHz
  * Caché: 12MB
  * TDP: 45W
  * Gráficos: [UHD Intel P630](https://www.intel.es/content/www/es/es/support/products/98913/graphics/graphics-for-7th-generation-intel-processors/intel-hd-graphics-p630.html)
* Memoria SDRAM DDR4 32GB (2 DIMM x 16GB 2RX8, 2666 MHz, sin ECC
* Disco duro SSD PCIe NVMe M.2 (clase 40) de 256 GB
* Pantalla Full HD UltraSharp:
  * Panel: LCD IPS de 15.6" antirreflectante
  * Resolución: 1920 x 1080
  * Gamut: 72% de la gama de colores
  * Sin funcionalidad táctil, sin WWAN, cámara de infrarrojos/micrófono
* Tarjeta gráfica [Radeon Pro WX 4150](https://www.amd.com/es/products/professional-graphics/radeon-pro-wx-4150-mobile) con memoria GDDR5 de 4 GB
* Teclado interno retroiluminado, español (QWERTY)
* Tarjeta Ethernet [Intel I219-LM](https://www.intel.com/content/www/us/en/products/network-io/ethernet/controllers/gigabit-ethernet-controllers/connection-i219-lm.html)
* Tarjeta Wi-Fi [Intel Wireless AC 9260](https://www.intel.com/content/www/us/en/products/wireless/wireless-products/dual-band-wireless-ac-9260.html) 802.11ac MU-MIMO Dual Band 2x2 + Bluetooth 5.0
* Batería 6 celdas 97Wh Lithium Ion con ExpressCharge (part number GW0K9)
* Cargador E5 180W 7,4mm Lot 6 PCR, Chicony, Cable de alimentación E5 (europeo)
* Sistema Operativo Ubuntu Linux 16.04

Se puede obtener información del equipo mediante **WMI** o **CIM**:

```
Get-CimInstance -ClassName Win32_ComputerSystem | Format-List Manufacturer, Model, SystemType
Get-CimInstance -ClassName Win32_ComputerSystemProduct | Format-List IdentifyingNumber, Vendor
Get-CimInstance -ClassName Win32_BIOS | Format-List SMBIOSBIOSVersion, Manufacturer, Name, SerialNumer
```

# Parámetros de la BIOS

La configuración de la BIOS (se entra con `F2`) se ha relizado mediante los siguientes parámetros a partir de los **BIOS Defaults**:

* General
  * Boot Sequence
    * UEFI: PM981 NVMe Samsung 256GB, Partition 1 &rarr; Windows Boot Manager

* System Configuration
  * Integrated NIC
    * Enable UEFI Network Stack &rarr; Enabled
    * Enabled
  * SMART Reporting
    * Enable SMART Reporting &rarr; Enabled
  * Keyboard Backlight Timeout on AC
    * 10 seconds &rarr; 30 seconds
  * Miscellaneous Devices
    * Enable Hard Drive Free Fall Protection &rarr; Disabled

* Video
  * LCD Brightness
    * Brightness On Battery &rarr; 60%

* Security
  * Password Change
    * Allow Non-Admin Password Changes &rarr; Disabled
  * OROM Keyboard Access
    * Disabled

* Power Management
  * USB Wake Support
    * Wake on Dell USB-C Dock &rarr; Disabled

* POST Behavior
  * Extend BIOS POST Time
    * 5 seconds

* Virtualization Support
  * Trusted Execution
    * Enabled

* SupportAssist System Resolution
  * Auto OS Recovery Threshold
    * OFF

# Instalación

| [Macrium Reflect Free](#macrium-reflect-free) | [Configuración Windows](#configuración-windows) | [Software](#software) |

La instalación del portátil se ha realizado mediante una [*OSBuild*](https://github.com/manelrodero/OSDBuilder/blob/main/Prod/Tasks/OSBuild%20Windows%2010%20Education%20x64%2020H2%20BLANK.json) de **Windows 10 Education x64 20H2** multilenguaje (`es-es`, `en-us` y `ca-es`).

Se ha realizado un particionado [**GPT/UEFI**](https://github.com/manelrodero/OSDBuilder/blob/main/Z-Extra/Diskpart_UEFI.txt) mediante el comando `diskpart.exe` desde una ventana de comandos (usando **Shift+F10** desde la pantalla de instalación).

Esta *OSBuild* utiliza un fichero [`AutoUnattend.xml`](https://github.com/manelrodero/OSDBuilder/blob/main/Z-Extra/AutoUnattend-Education.xml) que configura algunos aspectos de la instalación (idioma, zona horaria, cuenta principal, etc.).

También se copian [algunos scripts](https://github.com/manelrodero/OSDBuilder/tree/main/Share/Content/ExtraFiles/Windows%2010%20Scripts/Scripts) en `C:\Scripts` que pueden ser de ayuda para acabar la configuración del equipo.

* Cambiar la contraseña del usuario `Software`
* Forzar la interfaz en inglés mediante `C:\Scripts\Force-English.ps1`
* Configurar el nuevo Microsoft Edge mediante el asistente:
  * Tab page: Focused
  * Continue without signing-in

## Macrium Reflect Free

El primer programa que suelo instalar en un equipo es [Macrium Reflect Free](software#macrium-reflect-free) para poder hacer una copia de seguridad del mismo.

Después de instalarlo y configurarlo, se hace un primer **Backup** en `D:\EQUIPS\Macrium\Dell-Precision-7530-01-PreDrivers-00-00.mrimg` para poder volver a este punto si hay algún problema con la instalación de los drivers.

## Drivers

{: .box-note}
**Nota**: Después de muchas pruebas, se ha decidido realizar la [instalación automática](#instalación-automática) de los drivers usando **Dell Command Update**.

### Instalación manual

A continuación, se ha procedido a [instalar manualmente los drivers](#instalación-manual-de-drivers) siguiendo el [orden recomendado por Dell para un Precision 7530](https://www.dell.com/support/kbdoc/en-us/000144599/precision-7530-windows-10-driver-install-order):

* Chipset
* SATA
* Video
* Audio
* Network
* Security
* Input

### Instalación automática

La instalación automática de drivers en un equipo Dell se realiza mediante la aplicación [Dell Command Update](https://www.dell.com/support/kbdoc/en-us/000177325/dell-command-update) (DCU). Esta aplicación permite actualizar la BIOS, el firmware, los controladores y las aplicaciones desde una interfaz gráfica o mediante la línea de comands (`dcu-cli.exe`).

* [Dell Command \| Update Application for Windows 10](https://www.dell.com/support/home/en-us/drivers/DriversDetails?driverId=PF5GJ)
* [Dell Command \| Update Version 4.x User's Guide](https://dl.dell.com/topicspdf/command-update_users-guide_en-us.pdf)
* [Dell Command \| Update Version 4.x Reference Guide](https://dl.dell.com/topicspdf/command-update_reference-guide_en-us.pdf)

Se descarga el siguiente fichero:

| Fichero | [`Dell-Command-Update-Application-for-Windows-10_PF5GJ_WIN_4.0.0_A00.EXE`](https://dl.dell.com/FOLDER06747799M/1/Dell-Command-Update-Application-for-Windows-10_PF5GJ_WIN_4.0.0_A00.EXE) |
| Versión | 4.0.0, A00 |
| Categoría | Systems Management |
| Release date | 05 Nov 2020 |
| Last updated | 07 Jan 2021 |

Después se instala de forma silenciosa usando el modificador `/S` y se **reinicia** el equipo con `Restart-Computer`.

Una vez reiniciado, se ejecuta **Dell Command Update** y se elige la opción para reinstalar complemente la librería de drivers del equipo. DCU realiza la siguientes operaciones:

* Descarga la librería de drivers ([_driver pack_](#driver-pack)) para este modelo y _service tag_
* Extrae los drivers que contiene (a fecha 10 de enero de 2021 hay 25 drivers)
* Crea un punto de restauración
* Instala en el siguiente orden:
  * Intel Management Engine Components Installer (1932.12.0.1298)
  * Intel Dynamic Platform and Thermal Framework (8.4.10501.6067)
  * STMicro Accelerometer for Free Fall Data Protection (4.10.92)
  * Intel Chipset Device Software (10.1.17541.8066)
  * Realtek Memory Card Reader Driver (10.0.16299.21305)
  * Intel Serial IO Driver (30.100.1727.1)
  * Intel HID Event Filter Driver (2.2.1.372)
  * Intel Thunderbolt Controller Driver (17.4.79.510)
  * Dell Touchpad Driver (10.3201.101.215)
  * Intel Rapid Storage Technology and Management Console (17.5.9.1040)
  * AMD Radeon Pro WX4150 and WX7100 Graphics Driver (25.20.15026.5006)
  * Realtek IR Camera Driver (10.0.16299.20038)
  * Dell ControlVault2 Driver and Firmware (4.12.5.8)
  * Dell USB Smartcard Keyboard Driver (4.1.4.1)
  * Realtek High Definition Audio Driver (6.0.8838.1)
  * DW5811e_Customer Kit Module_Qualcomm Snapdragon X7 LTE Firmware and GNSS Driver (7.54.4799.502)
  * Intel UHD Graphics 600 P600 series Modern Driver (25.20.100.6519)
  * Qualcomm QCA61x4A/QCA6174A-XR/QCA9377 WiFi and Bluetooth Driver (12.0.0.916)
  * Intel PCIe Ethernet Controller Driver (24.1.0.0)
  * nVidia Quadro P Series Graphics Driver (26.21.14.3123)
  * Intel 9560/9260/18265/8265/7265/3165 WiFi Driver (21.40.1.2307)
  * ASMedia USB eXtensible Host Controller Driver (1.16.59.1)
  * Intel 9560/9260/18265/8265/7265/3165 Bluetooth UWD Driver (21.40.1.1)
  * Realtek USB GBE Ethernet Controller Driver (2.45.2019.0618)
  * Realtek USB Audio Driver (6.3.9600.2215)
* Reinicia el equipo

A continuación, se vuelve a ejecutar DCU y se elige la opción `CHECK` para comprobar actualizaciones del sistema. A fecha 10 de enero de 2021 hay 10 actualizaciones:

* 3 críticas
  * Dell Security Advisory Update - DSA-2020-059
  * Intel 9560/9260/18265/8265/7265/3165 Bluetooth UWD Driver
  * Intel 9560/9260/18265/8265/7265/3165 WiFi Driver
* 7 recomendadas
  * AMD Radeon Pro Graphics Driver
  * Intel HID Event Filter Driver
  * Intel Management Engine Components Installer
  * Intel PCIe Ethernet Controller Driver
  * Intel Serial IO Driver
  * Intel UHD Graphics Driver
  * STMicro Accelerometer for Free Fall Data Protection

 DCU realiza la siguientes operaciones:

* Descarga las 10 actualizaciones seleccionadas para instalar
* Crea un punto de restauración
* Instala en el siguiente orden:
  * Dell Security Advisory Update - DSA-2020-059 (1.0.0.0)
  * Intel HID Event Filter Driver (2.2.1.377)
  * Intel 9560/9260/8265/7265/3165 Wi-Fi Driver (21.110.2.1)
  * Intel 9560/9260/8265/7265/3165 Bluetooh UWD Driver (21.110.0.3)
  * AMD Radeon Pro Graphics Driver (26.20.13028.3001)
  * Intel UHD Graphics Driver (27.20.100.8280)
  * STMicro Accelerometer for Free Fall Data Protection (4.10.90) &rarr; Failed
  * Intel Management Engine Components Installer (2027.14.0.1683)
  * Intel Serial IO Driver (30.100.2020.7)
  * Intel PCIe Ethernet Controller Driver (24.1.0.0)
* Reinicia el equipo

{: .box-note}
**Nota**: Si se quisiera automatizar la instalación usando `dcu-cli.exe` se podrían usar scripts como los siguientes: [BruceSa85](https://github.com/brucesa85/Dell/blob/master/DellCommandUpdate.ps1), [PowershellBacon](https://github.com/PowershellBacon/Dell-Driver-Updates/blob/master/Update-Dell-Drivers.ps1).

A continuación se hace un **Backup** con **Macrium Reflect Free** en `D:\EQUIPS\Macrium\Dell-Precision-7530-02-PostDrivers-00-00.mrimg`.

## Configuración Windows

La configuración final de Windows es una mezcla de los parámetros incluidos en la imagen WIM y la ejecución de diferentes scripts ubicados en `C:\Scripts`:

* Deshabilitar drivers desde Windows Update `ExcludeWUDriversInQualityUpdate` (Offline WIM)
* Activar Windows (se activa automáticamente con licencia digital)
* Opciones del Explorador de Archivos (Offline WIM)
* Deshabilitar AutoPlay (Offline WIM)
* Deshabilitar Internet Explorer (Offline WIM)
* Deshabilitar Cortana (Offline WIM)
* Preferir IPv4 en lugar de IPv6 (AutoUnattend.xml)
* Configurar variable `DIRMCD` (AutoUnattend.xml)
* Deshabilitar Hibernation (AutoUnattend.xml)
* Mostrar dispositivos no presentes en el Administrador de dispositivos (AutoUnattend.xml)
* Añadir `D:\SOFT\Bin` a la variable PATH del sistema
* Habilitar ICMP v4 mediante `C:\Scripts\Enable-ICMPv4.ps1`
* Habilitar SMB mediante `C:\Scripts\Enable-SMB.ps1`
* Deshabilitar Netbios mediante `C:\Scripts\Disable-Netbios.ps1`
* Permitir la creación de *symbolic links* mediante `C:\Scripts\Config-SymLinks.ps1`
* Renombrar el equipo mediante `C:\Scripts\Rename-PC.ps1`
* Crear usuarios del equipo mediante `C:\Scripts\Create-Users.ps1` (utiliza el fichero `C:\Scripts\%ComputerName%.users`)
* Accesos directo a Shutdown, Restart y Logoff mediante `C:\Scripts\Copy-Icons.ps1`
* Establecer la imagen de cada usuario desde `C:\Users\Public\Pictures`
* Desactivar el inicio de sesión automático para terminar de configurar el dispositivo en cada usuario
* Establecer el wallpaper de la pantalla de bloqueo mediante `C:\Scripts\Config-LockScreenWallpaper.ps1`
* Actualizar ayuda de PowerShell mediante el *cmdlet* `Update-Help -Force`
* Recibir actualizaciones de otros productos Microsoft mediante `C:\Scripts\Config-MicrosoftUpdate.ps1`
* Habilitar Hyper-V mediante `C:\Scripts\Enable-HyperV.ps1` (reiniciará el equipo)

## Software

Las aplicaciones que se han instalado en este equipo usando los instaladores en `D:\SOFT\INSTALADORES` son las siguientes:

* [BIG-IP Edge Client](software#big-ip-edge-client)
* [Office Professional Plus 2019](software#office-professional-plus-2019)
* [Eduroam](software#eduroam)
* [Docker Desktop](software#docker-desktop)
* [VMware Workstation Pro](software#vmware-workstation-pro)
* [Oracle VM VirtualBox](software#oracle-vm-virtualbox)
* [DNIe](software#dnie)
* [Gemalto IDGo 800](software#gemalto-idgo-800)
* [7-Zip](software#7-zip)

# Instalación manual de drivers

| [BIOS](#bios) | [Chipset](#chipset) | [Serial ATA](#serial-ata) | [Video](#video) | [Audio](#audio) | [Network](#network) | [Security](#security) | [Mouse, Keyboard & Input Devices](#mouse-keyboard--input-devices) | [Docks/Stands](#docksstands) | [Other](#other) |

<p></p>

| [Exportación](#exportación) | [PnPutil.exe](#pnputilexe) | [Driver Pack](#driver-pack) | [Samsung 970 EVO Plus](#samsung-970-evo-plus) |

{: .box-note}
**Nota**: En la página de [soporte del Precision 7530](https://www.dell.com/support/home/en-us/product-support/product/precision-15-7530-laptop/overview) se puede introducir el **Service Tag** para encontrar los drivers específicos de mi equipo.

En un **Windows 10** recién instalado se recomienda instalar primero los drivers del **chipset de Intel** y, a continuación, el resto de drivers. El [orden de los drivers para un Precision 7530](https://www.dell.com/support/kbdoc/en-us/000144599/precision-7530-windows-10-driver-install-order) es el siguiente:

* Chipset
* SATA
* Video
* Audio
* Network
* Security
* Input

{: .box-note}
**Nota**: Los logs de la instalación se guardan en `C:\ProgramData\Dell\UpdatePackage\Log`. Se pueden borrar los directorios temporales en `C:\ProgramData\Dell\drivers`.

Para saber **qué cambios realiza la instalación de un driver**, se puede compara la salida del siguiente comandos antes y después de la instalación:

```
Get-CimInstance -ClassName Win32_PnPSignedDriver | Select-Object DeviceName, DriverVersion, Manufacturer, DeviceID | Sort-Object -Property DeviceID
```

## BIOS

Es interesante comprobar si la BIOS del equipo está actualizada para solucionar posibles problemas de **seguridad** o de **rendimiento**.

### 00) Dell Precision 7530 and 7730 System BIOS

| Fichero | `Precision_7x30_1.14.4.exe` |
| Versión | 1.14.4 |
| Categoría | BIOS |
| Release date | 13 Nov 2020 |
| Last updated | 13 Nov 2020 |

{: .box-warning}
**Warning**: Restart Required

## Chipset

### 01) Intel Chipset Device Software

* Se instala en `C:\Program Files)\Intel\Intel(R) Chipset Device Software`

| Fichero | `Intel-Chipset-Device-Software_5MPRF_WIN_10.1.18121.8164_A09.EXE` |
| Versión | 10.1.18121.8164, A09 |
| Categoría | Chipset |
| Release date | 25 Mar 2020 |
| Last updated | 17 Dec 2020 |
| Installer | 15 Oct 2019 |

{: .box-warning}
**Warning**: Restart Required

Los cambios que ha habido en los drivers de los dispositivos son:

| Dispositivo | Versión | ID | Nuevo Dispositivo | Versión | Fabricante |
| --- | --- | --- | --- | --- |
| PCI Device | | PCI\VEN_8086&DEV_A324 | Intel(R) SPI (flash) Controller - A324 | 10.1.16.7 | Intel |
| SM Bus Controller | | PCI\VEN_8086&DEV_A323 | Intel(R) SMBus - A323 | 10.1.16.7 | Intel |
| PCI standard ISA bridge | 10.0.19041.1 | PCI\VEN_8086&DEV_A30E | C240 Series Chipset Family LPC Controller (CM246) - A30E | 10.1.16.7 | Intel |
| PCI Express Root Port | 10.0.19041.662 | PCI\VEN_8086&DEV_A330 | Intel(R) PCI Express Root Port #9 - A330 | 10.1.16.7 | Intel |
| PCI Express Root Port | 10.0.19041.662 | PCI\VEN_8086&DEV_A33E | Intel(R) PCI Express Root Port #7 - A33E | 10.1.16.7 | Intel | 
| PCI Express Root Port | 10.0.19041.662 | PCI\VEN_8086&DEV_A33D | Intel(R) PCI Express Root Port #6 - A33D | 10.1.16.7 | Intel |
| PCI Express Root Port | 10.0.19041.662 | PCI\VEN_8086&DEV_A338 | Intel(R) PCI Express Root Port #1 - A338 | 10.1.16.7 | Intel |
| PCI Express Root Port | 10.0.19041.662 | PCI\VEN_8086&DEV_A32C | Intel(R) PCI Express Root Port #21 - A32C | 10.1.16.7 | Intel |
| PCI Simple Communications Controller | | PCI\VEN_8086&DEV_A360 | Intel(R) Management Engine Interface - A360 | 0.0.0.1 | Intel |
| PCI Data Acquisition and Signal Processing Controller| | PCI\VEN_8086&DEV_A379 | Intel(R) Thermal Subsystem - A379 | 10.1.16.7 | Intel |
| Base System Device | | PCI\VEN_8086&DEV_1911 | Intel(R) Gaussian Mixture Model - 1911 | 10.1.7.3 | Intel |
| PCI Data Acquisition and Signal Processing Controller | | PCI\VEN_8086&DEV_1903 | Intel(R) Processor Power and Thermal Controller - 1903 | 10.1.7.3 | Intel |
| PCI Express Root Port | 10.0.19041.662 | PCI\VEN_8086&DEV_1901 | Intel(R) PCIe Controller (x16) - 1901 | 10.1.7.3 | Intel |
| PCI standard host CPU bridge | 10.0.19041.1 | PCI\VEN_8086&DEV_3EC4 | Intel(R) Host Bridge/DRAM Registers - 3EC4 | 10.1.14.7 | Intel |

### 02) Intel Management Engine Components Installer

{: .box-note}
**Nota**: Se instala primero el driver "Intel Dynamic Platform and Thermal Framework" porque es más antiguo que éste

* Se instala en `C:\Program Files (x86)\Intel\Intel(R) Management Engine Components` y en `C:\Program Files\Intel\Intel(R) Management Engine Components`

| Fichero | `Intel-Management-Engine-Components-Installer_M3H6W_WIN_2027.14.0.1683_A05_01.EXE` |
| Versión | 2027.14.0.1683, A05 |
| Categoría | Chipset |
| Release date | 28 Jul 2020 |
| Last updated | 10 Nov 2020 |
| Installer | 27 Jul 2020 |

Los cambios que ha habido en los drivers de los dispositivos son:

| Dispositivo | Versión | ID | Nuevo Dispositivo | Versión | Fabricante |
| --- | --- | --- | --- | --- |
| Intel(R) Management Engine Interface - A360 | 0.0.0.1 | PCI\VEN_8086&DEV_A360 | Intel(R) Management Engine Interface | 2013.14.0.1529 | Intel |

### 03) Intel Dynamic Platform and Thermal Framework

* Se instala en `C:\Program Files\Intel\Intel(R) Dynamic Platform and Thermal Framework`
* Instala un "Intel(R) Dynamic Platform and Thermal Processor Participant" más antiguo que el instalado (10.1.7.3)

| Fichero | `Intel-Dynamic-Platform-and-Thermal-Framework_591DK_WIN_8.4.10501.6067_A01.EXE` |
| Versión | 8.4.10501.6067, A01 |
| Categoría | Chipset |
| Release date  | 05 Jul 2018  |
| Last updated  | 26 Nov 2019 |
| Installer | 26 Apr 2018 |

Los cambios que ha habido en los drivers de los dispositivos son:

| Dispositivo | Versión | ID | Nuevo Dispositivo | Versión | Fabricante |
| --- | --- | --- | --- | --- |
| | | ACPI\INT3400\2&DABA3FF&0 | Intel(R) Dynamic Platform and Thermal Framework Manager | 8.4.10501.6067 | Intel |
| | | ACPI\INT3403\DSC-GPU | Intel(R) Dynamic Platform and Thermal Framework Generic Participant | 8.4.10501.6067 | Intel |
| | | ACPI\INT3403\NGFF | Intel(R) Dynamic Platform and Thermal Framework Generic Participant | 8.4.10501.6067 | Intel |
| | | ACPI\INT3403\SKIN | Intel(R) Dynamic Platform and Thermal Framework Generic Participant | 8.4.10501.6067 | Intel |
| Intel(R) Processor Power and Thermal Controller - 1903 | 10.1.7.3 | PCI\VEN_8086&DEV_1903 | Intel(R) Dynamic Platform and Thermal Framework Processor Participant | 8.4.10501.6067 | Intel |

### 04) Intel Serial IO Driver

* Se instala en `C:\Program Files\Intel\Intel(R) Serial IO`

| Fichero | `Intel-Serial-IO-Driver_RDY6W_WIN_30.100.2020.7_A05` |
| Versión | 30.100.2020.7, A05 |
| Categoría | Chipset |
| Release date | 27 Oct 2020 |
| Last updated | 08 Dec 2020 |
| Installer | 14 Mar 2019 |

{: .box-warning}
**Warning**: Restart Required

Los cambios que ha habido en los drivers de los dispositivos son:

| Dispositivo | Versión | ID | Nuevo Dispositivo | Versión | Fabricante |
| --- | --- | --- | --- | --- |
| Intel(R) Serial IO GPIO Host Controller - INT3450 | 30.100.1816.3 | ACPI\INT3450\3&11583659&0 | Intel(R) Serial IO GPIO Host Controller - INT3450 | 30.100.2020.7 | Intel |
| Intel(R) Serial IO I2C Host Controller - A368 | 30.100.1929.1 | PCI\VEN_8086&DEV_A368 | Intel(R) Serial IO I2C Host Controller - A368 | 30.100.2020.7 | Intel |
| Intel(R) Serial IO I2C Host Controller - A369 | 30.100.1929.1 | PCI\VEN_8086&DEV_A369 | Intel(R) Serial IO I2C Host Controller - A369 | 30.100.2020.7 | Intel |

### 05) Intel HID Event Filter Driver

* En el orden de instalación estaba en esta posición porque se consideraba "Chipset", ahora se considera "Mouse, Keyboard & Input Devices"
* Se instala en `C:\Program Files (x86)\Intel\Intel(R) HID Event Filter`

| Fichero | `Intel-HID-Event-Filter-Driver_33CDY_WIN_2.2.1.377_A11_04.EXE` |
| Versión | 2.2.1.377, A11 |
| Categoría | Mouse, Keyboard & Input Devices |
| Release date | 30 Jul 2019 |
| Last updated | 14 Dec 2020 |
| Installer | 26 Jun 2019 |

{: .box-warning}
**Warning**: Restart Required

Los cambios que ha habido en los drivers de los dispositivos son:

| Dispositivo | Versión | ID | Nuevo Dispositivo | Versión | Fabricante |
| --- | --- | --- | --- | --- |
| | | ACPI\INT33D5\2&DABA3FF&0 | Intel(R) HID Event Filter | 2.2.1.377 | Intel |

### 06) Intel Thunderbolt Controller Driver

* Se instala en `C:\Program Files (x86)\Intel\Thunderbolt Software`

| Fichero | `Intel-Thunderbolt-Controller-Driver_TBT79_WIN_17.4.79.510_A12.EXE` |
| Versión | 17.4.79.510, A12 |
| Categoría | Chipset |
| Release date  | 06 Mar 2019  |
| Last updated  | 09 Aug 2019 |
| Installer | 22 Jan 2019 |

### 07a) STMicroelectronics Free Fall Data Protection Driver

* Se instala en `C:\Program Files\ST Microelectronics`
* Automáticamente se instala "Dell Free Fall Data Protection" desde la [Microsoft Store](https://www.microsoft.com/es-es/p/dell-free-fall-data-protection/9n0ddr0p6blg?rtc=1&activetab=pivot:overviewtab)
* Aunque el instalador no lo indica, se reinicia el equipo según las instrucciones

| Fichero | `STMicroelectronics-Free-Fall-Data-Protection-Driver_CCXN6_WIN_4.10.104_A00` |
| Versión | 4.10.104, A00 |
| Categoría | Chipset |
| Release date | 07 May 2020 |
| Last updated | 05 Aug 2020 |
| Installer | 19 Feb 2020 |

{: .box-warning}
**Warning**: Restart Required

Los cambios que ha habido en los drivers de los dispositivos son:

| Dispositivo | Versión | ID | Nuevo Dispositivo | Versión | Fabricante |
| --- | --- | --- | --- | --- |
| | | ACPI\SMO8810\1 | STMicroelectronics 3-Axis Digital Accelerometer | 2.2.7.2 | STMicroelectronics |

### 07b) Dell Free Fall Data Protection Application

* En principio, no haría falta porque el equipo no tiene disc duros mecánicos

| Fichero | `Dell-Free-Fall-Data-Protection-Application_FFAP1_WIN64_1.0.26.0_A05.EXE` |
| Versión | 1.0.26.0, A05 |
| Categoría | Chipset |
| Release date | 05 Aug 2020 |
| Last updated | 25 Aug 2020 |

### 08) Realtek PCIE Memory Card Reader Driver

* Se instala en `C:\Program Files (x86)\Realtek\Realtek Card Reader`

| Fichero | `Realtek-PCIE-Memory-Card-Reader-Driver_T5KCG_WIN_10.0.16299.21305_A01.EXE` |
| Versión | 10.0.16299.21305, A01 |
| Categoría | Chipset |
| Release date  | 08 May 2018  |
| Last updated  | 30 Aug 2019 |
| Installer | 12 Feb 2018 |

Los cambios que ha habido en los drivers de los dispositivos son:

| Dispositivo | Versión | ID | Nuevo Dispositivo | Versión | Fabricante |
| --- | --- | --- | --- | --- |
| PCI Device | | PCI\VEN_10EC&DEV_5260 | Realtek PCIE CardReader | 10.0.16299.21305 | Realtek Semiconductor Corp. |
| | | SWD\MSRRAS\{5E259276-BC7E-40E3-B93B-8F89B5F3ABC0} | Generic software device | 10.0.19041.1 | Microsoft |
| | | SWD\MSRRAS\MS_AGILEVPNMINIPORT | WAN Miniport (IKEv2) | 10.0.19041.1 | Microsoft |
| | | SWD\MSRRAS\MS_L2TPMINIPORT | WAN Miniport (L2TP) | 10.0.19041.1 | Microsoft |
| | | SWD\MSRRAS\MS_NDISWANBH | WAN Miniport (Network Monitor) | 10.0.19041.1 | Microsoft |
| | | SWD\MSRRAS\MS_NDISWANIP | WAN Miniport (IP) | 10.0.19041.1 | Microsoft |
| | | SWD\MSRRAS\MS_NDISWANIPV6 | WAN Miniport (IPv6) | 10.0.19041.1 | Microsoft |
| | | SWD\MSRRAS\MS_PPPOEMINIPORT | WAN Miniport (PPPOE) | 10.0.19041.1 | Microsoft |
| | | SWD\MSRRAS\MS_PPTPMINIPORT | WAN Miniport (PPTP) | 10.0.19041.1 | Microsoft |
| | | SWD\MSRRAS\MS_SSTPMINIPORT | WAN Miniport (SSTP) | 10.0.19041.1 | Microsoft |

### USB 3.0

{: .box-warning}
**Atención**: No hay drivers para descargar ya que, desde Windows 8.1, hay un driver nativo USB 3.0 instalado por defecto.

## Serial ATA

### 09a) Intel Rapid Storage Technology Driver and Management Console

{: .box-note}
**Nota**: Es mejor utilizar el paquete **09b** que únicamente incluye el driver y así no hay que preocuparse por la aplicación.

* No instalar el software, únicamente el [driver AHCI](https://www.win-raid.com/t25f23-Which-are-the-quot-best-quot-Intel-AHCI-RAID-drivers.html) extrayéndolo en `C:\Dell\RST`
* Actualizar *Controladora SATA AHCI estándar* usando `C:\Dell\RST\Drivers\Production\Windows10-x64`
* Aparece *Intel(R) 300 Series Chipset Family SATA AHCI Controller*

| Fichero | `Intel-Rapid-Storage-Technology-Driver-and-Management_KV2CM_WIN_17.5.9.1040_A04.EXE` |
| Versión | 17.5.9.1040, A04 |
| Categoría | Serial ATA |
| Release date | 25 Feb 2020 |
| Last updated | 16 Dec 2020 |
| Installer | 07 Jan 2020 |

{: .box-warning}
**Warning**: Restart Required

### 09b) Intel Rapid Storage Technology Driver

* Usa `C:\Program Files\DIFX\D29FE547208FE130\dpinst.exe` para instalar el controlador
* Aunque el instalador no lo indica, se reinicia el equipo según las instrucciones

| Fichero | `Intel-Rapid-Storage-Technology-Driver_CG84X_WIN64_17.5.9.1040_A04.EXE` |
| Versión | 17.5.9.1040, A04 |
| Categoría | Serial ATA |
| Release date | 02 Mar 2020 |
| Last updated | 19 Jun 2020 |
| Installer | 07 Jan 2020 |

{: .box-warning}
**Warning**: Restart Required

Los cambios que ha habido en los drivers de los dispositivos son:

| Dispositivo | Versión | ID | Nuevo Dispositivo | Versión | Fabricante |
| --- | --- | --- | --- | --- |
| Standard SATA AHCI Controller | 10.0.19041.488 | PCI\VEN_8086&DEV_A353 | Intel(R) 300 Series Chipset Family SATA AHCI Controller | 17.5.9.1040 | Intel Corporation |

## Video

### 10a) Intel UHD Graphics Driver

{: .box-note}
**Nota**: Se instala la versión [27.20.100.9079](https://downloadcenter.intel.com/download/30079/Intel-Graphics-Windows-10-DCH-Drivers) del 23 Dic 2020 descargada desde Intel: `10c-igfx_win10_100.9079.exe`.

* Intel recomienda usar la versión "modern" (UWD, Universal Windows Drivers) para Windows 10 1809+ y Windows Server 2019
* Antes también se podía descargar una versión "legacy" (23.20.16.4973)
* Ejecutará automáticamente `WinSAT.exe` y activará el tema de escritorio Windows Aero (si es compatible)
* La tarjeta gráfica integrada en el procesador es una *Intel(R) UHD Graphics P630*
* Se instala en `C:\Program Files\Intel\Media Resource`
* Automáticamente se instala "Intel Graphics Command Center" desde la [Microsoft Store](https://www.microsoft.com/en-us/p/intel-graphics-command-center/9plfnlnt3g5g?activetab=pivot:overviewtab)

| Fichero | `Intel-UHD-Graphics-Driver_CKCJR_WIN_27.20.100.8280_A09_02.EXE` |
| Versión | 27.20.100.8280, A09 |
| Categoría | Video |
| Release date | 17 Jul 2020 |
| Last updated | 20 Jul 2020 |
| Installer | 06 Jul 2020 |

{: .box-warning}
**Warning**: Restart Required

### 10b) Intel Graphics Command Center Application

* Este programa se ha instalado automáticamente después de instalar el driver anterior
* [Intel Graphics Command Center](https://www.intel.com/content/www/us/en/support/articles/000055840/graphics.html) ya no se incluye en los drivers [Declarative Componentized Hardware](https://www.intel.com/content/www/us/en/support/articles/000031275/graphics.html) (DCH)
* Los requisitos de este programa son:
  * 6th Generation Intel Core platforms or newer
  * Windows 10 Versión 1709 or higher
  * Windows 10 DCH Intel Graphics Driver Versión 25.20.100.6618 or newer

| Fichero | `Intel-Graphics-Command-Center-Application_GG57X_WIN64_1.100.2765.0_A08.EXE` |
| Versión | 1.100.2765.0, A08 |
| Categoría | Video |
| Release date | 15 Sep 2020 |
| Last updated | 07 Oct 2020 |

{: .box-warning}
**Warning**: Restart Required

### 10c) Intel Graphics Windows 10 DCH Drivers

{: .box-note}
**Nota**: Versión descargada desde [Intel](https://downloadcenter.intel.com/download/30079/Intel-Graphics-Windows-10-DCH-Drivers).

| Fichero | `10c-igfx_win10_100.9079.exe` |
| Versión | 27.20.100.9079 |
| Release date | 23 Dic 2020 |

Los cambios que ha habido en los drivers de los dispositivos son:

| Dispositivo | Versión | ID | Nuevo Dispositivo | Versión | Fabricante |
| --- | --- | --- | --- | --- |
| High Definition Audio Device | 10.0.19041.264 | HDAUDIO\FUNC_01&VEN_8086&DEV_280B | Sonido Intel(R) para pantallas | 10.27.0.9 | Intel(R) Corporation |
| Microsoft Basic Display Adapter | 10.0.19041.1 | PCI\VEN_8086&DEV_3E94 | Intel(R) UHD Graphics P630 | 27.20.100.9079 | Intel Corporation |
| | | SWD\DRIVERENUM\CUI&4&39276A97&0 | Intel(R) Graphics Control Panel | 27.20.100.9079 | Intel Corporation |
| | | SWD\DRIVERENUM\IGCC&4&39276A97&0 | Intel(R) Graphics Command Center | 27.20.100.9079 | Intel Corporation |

### 11a) AMD Radeon Pro Graphics Driver

{: .box-note}
**Nota**: Se instala la versión [20.Q4 27.20.11027.5002](https://www.amd.com/es/support/professional-graphics/mobile-platforms/radeon-pro-wx-x100-mobile-series/radeon-pro-wx-4150) del 27 Oct 2020 descargada desde AMD: `11c-win10-radeon-pro-software-enterprise-20.q4-nov10.exe`.

* Se recomienda instalar el driver más actual, aunque no esté certificado ISV
* Instalación personalizada &rarr; Todo
* Borrar directorio huérfano `C:\AMD`

| Fichero | `AMD-Radeon-Pro-Graphics-Driver_9JC83_WIN_26.20.13028.3001_A05.EXE` |
| Versión | 26.20.13028.3001, A05 |
| Categoría | Video |
| Release date | 01 Jun 2020 |
| Last updated | 01 Jun 2020 |
| Installer | 21 Aug 2019 |

{: .box-warning}
**Warning**: Restart Required

### 11b) ISV Certified Driver - AMD Mobile workstation

* Se recomienda instalar el driver más actual, aunque no esté certificado ISV

| Fichero | `ISV-Certified-Driver-AMD-Mobile-workstation_DNHN8_WIN_25.20.15019.9_A00.EXE` |
| Versión | 25.20.15019.9, A00 |
| Categoría | Video |
| Release date  | 14 May 2019  |
| Last updated  | 23 Aug 2019 |

{: .box-warning}
**Warning**: Restart Required

### 11c) Radeon Pro Software (Adrenaline 2020 Edition)

{: .box-note}
**Nota**: Versión descargada desde [AMD](https://www.amd.com/es/support/professional-graphics/mobile-platforms/radeon-pro-wx-x100-mobile-series/radeon-pro-wx-4150).

* Instalación personalizada &rarr; Todo
* Borrar directorio huérfano `C:\AMD`

| Fichero | `11c-win10-radeon-pro-software-enterprise-20.q4-nov10.exe` |
| Versión | 20.Q4 27.20.11027.5002 |
| Release date | 27 Oct 2020 |

Los cambios que ha habido en los drivers de los dispositivos son:

| Dispositivo | Versión | ID | Nuevo Dispositivo | Versión | Fabricante |
| --- | --- | --- | --- | --- |
| PCI Express Root Complex | 10.0.19041.662 | ACPI\PNP0A08\0 | Pci Bus | 20.10.0.0 | AMD |
| High Definition Audio Device | 10.0.19041.264 | HDAUDIO\FUNC_01&VEN_1002&DEV_AA01 | | |
| Microsoft Basic Display Adapter | 10.0.19041.1 | PCI\VEN_1002&DEV_67E8 | Radeon (TM) Pro WX 4150 Graphics | 27.20.11027.5002 | Advanced Micro Devices, Inc. |
| High Definition Audio Controller | 10.0.19041.1 | PCI\VEN_1002&DEV_AAE0 | | |
| | | ROOT\AMDLOG\0000 | AMD LOG | 20.40.0.3 | AMD |

## Audio

### 12a) Realtek High Definition Audio Driver

* Automáticamente se instala "Waves MaxxAudio Pro for Dell" desde la [Microsoft Store](https://www.microsoft.com/es-es/p/waves-maxxaudio-pro-for-dell/9nb9srtl2kpt?rtc=1&activetab=pivot:overviewtab#)

| Fichero | `Realtek-High-Definition-Audio-Driver_MHXXD_WIN_6.0.8838.1_A10.EXE` |
| Versión | 6.0.8838.1, A10 |
| Categoría | Audio |
| Release date | 08 Jan 2020 |
| Last updated | 21 Jul 2020 |
| Installer | 18 Nov 2019 |

{: .box-warning}
**Warning**: Restart Required

Los cambios que ha habido en los drivers de los dispositivos son:

| Dispositivo | Versión | ID | Nuevo Dispositivo | Versión | Fabricante |
| --- | --- | --- | --- | --- |
| High Definition Audio Device | 10.0.19041.264 | HDAUDIO\FUNC_01&VEN_10EC&DEV_0289 | Realtek Audio | 6.0.8838.1 | Microsoft |
| | | SWD\DRIVERENUM\{3ED78669-04CF-413C-A8DF-553436CEA20F}#WAVESAPO&5&1562004D&0 | Waves APO | 3.2.0.86 | Waves |
| | | SWD\DRIVERENUM\{C3A63EDD-2D27-4B66-B155-5E94B43D926A}#REALTEKAPO&5&1562004D&0 | Realtek Audio Effects Component | 11.0.6000.733 | Realtek |
| | | SWD\DRIVERENUM\{C3A63EDD-2D27-4B66-B155-5E94B43D926A}#REALTEKASIO&5&1562004D&0 | Realtek Asio Component | 1.0.0.5 | Realtek |
| | | SWD\DRIVERENUM\{C3A63EDD-2D27-4B66-B155-5E94B43D926A}#REALTEKSRV&5&1562004D&0 | Realtek Audio Universal Service | 1.0.0.217 | Realtek |

### 12b) Waves MaxxAudio Pro Application

* Este programa se ha instalado automáticamente después de instalar el driver anterior

| Fichero | `Waves-MaxxAudio-Pro-Application_VWPYY_WIN64_1.1.131.0_A07.EXE` |
| Versión | 1.1.131.0, A07 |
| Categoría | Audio |
| Release date| 10 Aug 2020|
| Last updated| 14 Sep 2020 |

## Network

### 13a) Intel PCIe Ethernet Controller Driver

{: .box-note}
**Nota**: Se instala la versión [25.6 12.19.1.32](https://downloadcenter.intel.com/product/82185/Intel-Ethernet-Connection-I219-LM) del 18 Dic 2020 descargada desde Intel: `13b-PROWinx64.exe`.

* Actualiza el driver Intel Ethernet Connection I219 de la 12.17.10.8 &rarr; 12.18.9.10
* Aunque el instalador no lo indica, se reinicia el equipo según las instrucciones

| Fichero | `Intel-PCIe-Ethernet-Controller-Driver_VP20T_WIN_24.1.0.0_A13_01.EXE` |
| Versión | 24.1.0.0, A13 |
| Categoría | Network |
| Release date | 15 Nov 2019 |
| Last updated | 03 Aug 2020 |
| Installer | 12 Nov 2019 |

{: .box-warning}
**Warning**: Restart Required

### 13b) Intel Ethernet Connection I219-LM

{: .box-note}
**Nota**: Versión descargada desde [Intel](https://downloadcenter.intel.com/product/82185/Intel-Ethernet-Connection-I219-LM).

* Instalación completa (incluyendo PROSet)

| Fichero | `13b-PROWinx64.exe` |
| Versión | 25.6 12.19.1.32 |
| Release date | 18 Dic 2020 |

Los cambios que ha habido en los drivers de los dispositivos son:

| Dispositivo | Versión | ID | Nuevo Dispositivo | Versión | Fabricante |
| --- | --- | --- | --- | --- |
| Intel(R) Ethernet Connection (7) I219-LM | 12.17.10.8 | PCI\VEN_8086&DEV_15BB | Intel(R) Ethernet Connection (7) I219-LM | 12.19.1.32 | Intel |

### 14a) Intel 9560/9260/8265/7265/3165 WiFi Driver

{: .box-note}
**Nota**: Se instala la versión [22.10.0.7](https://downloadcenter.intel.com/product/99445/Intel-Wireless-AC-9260) del 1 Dic 2020 descargada desde Intel: `14b-WiFi_22.10.0_Driver64_Win10.zip`.

* Instalación personalizada
* Instalar únicamente el controlador. [Intel PROSet/Wireless está *deprecated*](https://www.intel.com/content/www/us/en/support/articles/000031026/network-and-io/wireless.html)
* Actualiza el driver Intel Wireless-AC 9260 160MHz de la 21.10.2.2 &rarr; 21.110.2.1

| Fichero | `Intel-9560-9260-8265-7265-3165-Wi-Fi-Driver_39G3R_WIN_21.110.2.1_A26_04.EXE` |
| Versión | 21.110.2.1, A26 |
| Categoría | Network |
| Release date| 07 Sep 2020|
| Last updated| 29 Dec 2020 |

### 14b) Intel Wireless AC 9260

**Nota**: Versión descargada desde [Intel](https://downloadcenter.intel.com/product/99445/Intel-Wireless-AC-9260).

* Descomprimir el fichero ZIP
* Actualizar el driver Intel Wireless-AC 9260 160MHz de la 21.10.2.2 &rarr; 22.10.0.7

| Fichero | `14b-WiFi_22.10.0_Driver64_Win10.zip` |
| Versión | 22.10.0.7 |
| Release date | 1 Dic 2020 |

Los cambios que ha habido en los drivers de los dispositivos son:

| Dispositivo | Versión | ID | Nuevo Dispositivo | Versión | Fabricante |
| --- | --- | --- | --- | --- |
| Intel(R) Wireless-AC 9260 160MHz | 21.10.2.2 | PCI\VEN_8086&DEV_2526 | Intel(R) Wireless-AC 9260 160MHz | 22.10.0.7 | Intel Corporation |

### 15) Intel 9560/9260/8265/7265/3165 Bluetooth UWD Driver

* Instalación completa

| Fichero | `Intel-9560-9260-8265-7265-3165-Bluetooth-UWD-Driver_52Y48_WIN_21.110.0.3_A30_04.EXE` |
| Versión | 21.110.0.3, A30 |
| Categoría | Network |
| Release date| 07 Sep 2020|
| Last updated| 29 Dec 2020 |
| Installer | 04 Sep 2020 |

Los cambios que ha habido en los drivers de los dispositivos son:

| Dispositivo | Versión | ID | Nuevo Dispositivo | Versión | Fabricante |
| --- | --- | --- | --- | --- |
| | | {2F2B7B01-597A-434C-8DD6-D27CD46EF73C}\IBTUSBFILTER\6&2FA275E4&0&00 | IBtUsb_Filter_00 | | |
| | | BTH\MS_BTHBRB\6&2FA275E4&0&1 | Microsoft Bluetooth Enumerator | 10.0.19041.662 | Microsoft |
| | | BTH\MS_BTHLE\6&2FA275E4&0&3 | Microsoft Bluetooth LE Enumerator | 10.0.19041.488 | Microsoft |
| | | BTH\MS_BTHPAN\6&2FA275E4&0&2 | Bluetooth Device (Personal Area Network) | 10.0.19041.1 | Microsoft |
| | | BTH\MS_RFCOMM\6&2FA275E4&0&0 | Bluetooth Device (RFCOMM Protocol TDI) | 10.0.19041.1 | Microsoft |
| | | SWD\RADIO\BLUETOOTH_48F17FF74986 | Generic software device | 10.0.19041.1 | Microsoft |
| | | USB\VID_8087&PID_0025\5&3B9D3DC&0&14 | Intel(R) Wireless Bluetooth(R) | 21.110.0.3 | Intel Corporation |

## Security

### 17) Dell ControlVault2 Driver and Firmware

{: .box-note}
**Nota**: Dell ControlVault is a hardware-based security solution that provides a secure bank for storing and processing user credentials. ControlVault keeps passwords, biometric templates, and security codes within firmware.

| Fichero | `Dell-ControlVault2-Driver-and-Firmware_N23KC_WIN64_4.12.5.8_A21.EXE` |
| Versión | 4.12.5.8, A21 |
| Categoría | Security |
| Release date | 14 Feb 2020 |
| Last updated | 29 Jun 2020 |
| Installer | 26 Dic 2019 |

{: .box-warning}
**Warning**: Restart Required

Los cambios que ha habido en los drivers de los dispositivos son:

| Dispositivo | Versión | ID | Nuevo Dispositivo | Versión | Fabricante |
| --- | --- | --- | --- | --- |
| Broadcom USH w/touch sensor | | USB\VID_0A5C&PID_5833&MI_00\6&1FF64837&0&0000 | Dell ControlVault w/ Fingerprint Touch Sensor | 4.12.3.5 | Dell |
| Broadcom USH | | USB\VID_0A5C&PID_5833&MI_03\6&1FF64837&0&0003 | Control Vault w/ Fingerprint Touch Sensor | 4.12.3.2 | Broadcom Corporation |

## Mouse, Keyboard & Input Devices

### 18) Realtek IR Camera Driver

* Cambiará el driver "Integrad Webcam" de 10.0.19041.488 a 10.0.16299.20038
* Se comprueba que la cámara es compatible con **Windows Hello**

| Fichero | `Realtek-IR-Camera-Driver_8X9XJ_WIN_10.0.16299.20038_A12.EXE` |
| Versión | 10.0.16299.20038, A12 |
| Categoría | Mouse, Keyboard & Input Devices |
| Release date  | 05 Jul 2018  |
| Last updated  | 22 Apr 2020 |
| Installer | 16 May 2018 |

{: .box-warning}
**Warning**: Restart Required

Los cambios que ha habido en los drivers de los dispositivos son:

| Dispositivo | Versión | ID | Nuevo Dispositivo | Versión | Fabricante |
| --- | --- | --- | --- | --- |
| | | ROOT\WINDOWSHELLOFACESOFTWAREDRIVER\0000 | Windows Hello Face Software Device | 10.0.19041.662 | Windows Hello Face |
| USB Video Device | 10.0.19041.488 | USB\VID_0BDA&PID_58F6&MI_00\6&34E80799&0&0000 | Realtek DMFT - RGB | 10.0.16299.20038 | Realtek |
| USB Video Device | 10.0.19041.488 | USB\VID_0BDA&PID_58F6&MI_02\6&34E80799&0&0002 | Realtek DMFT - IR | 10.0.16299.20038 | Realtek |

### 19a) Dell Touchpad Driver

| Fichero | `Dell-Touchpad-Driver_2NNNK_WIN_10.3201.101.215_A09.EXE` |
| Versión | 10.3201.101.215, A09 |
| Categoría | Mouse, Keyboard & Input Devices |
| Release date  | 20 Apr 2020 |
| Last updated  | 20 Apr 2020 |

{: .box-warning}
**Warning**: Restart Required

Los cambios que ha habido en los drivers de los dispositivos son:

| Dispositivo | Versión | ID | Nuevo Dispositivo | Versión | Fabricante |
| --- | --- | --- | --- | --- |
| HID-compliant vendor-defined device | 10.0.19041.1 | HID\DELL0831&COL03\5&93EBAC2&0&0002 | Dell Touchpad | 10.3201.101.215 | ALPSALPINE |
| | | HID\VID_044E&PID_1212&COL01&COL01\7&622E91C&0&0000 | HID-compliant mouse | 10.0.19041.1 | Microsoft |
| | | HID\VID_044E&PID_1212&COL01&COL02\7&622E91C&0&0001 | HID Keyboard Device | 10.0.19041.1 | |
| | | HID\VID_044E&PID_1212&COL01\6&1417E5DD&0&0000 | AlpsAlpine Virtual HID Device | 10.0.0.126 | AlpsAlpine |

### 19b) Dell Touchpad Assistant Application

* En principio, no haría falta porque no utilizo los gestos del touchpad
* La aplicación "Dell Touchpad Assistant" se puede instalar desde la [Microsoft Store](https://www.microsoft.com/es-es/p/dell-touchpad-assistant/9n5gllfvs68r?rtc=1&activetab=pivot:overviewtab)

| Fichero | `Dell-Touchpad-Assistant-Application_6K88F_WIN64_1.1.9.0_A04.EXE` |
| Versión | 1.1.9.0, A04 |
| Categoría | Mouse, Keyboard & Input Devices |
| Release date  | 07 Aug 2020 |
| Last updated  | 10 Aug 2020 |

## Docks/Stands

### 16) Realtek USB GBE Ethernet Controller Driver

* No lo instalo porque parecen pensados para los docks o el adaptador de USB a GbE
* Antes estaba en la categoría "Network"

| Fichero | `Realtek-USB-GBE-Ethernet-Controller-Driver_7VDGG_WIN_2.45.2020.0601_A20_02.EXE` |
| Versión | 2.45.2020.0601, A20 |
| Categoría | Docks/Stands |
| Release date | 15 Jul 2020 |
| Last updated | 29 Dec 2020 |

{: .box-warning}
**Warning**: Restart Required

### 20) Realtek USB Audio Driver

* No lo instalo porque es el códec de audio USB de Realtek ALC3263 para las docks

| Fichero | `Realtek-USB-Audio-Driver_RRPCC_WIN_6.3.9600.2250_A16.EXE` |
| Versión | 6.3.9600.2250, A16 |
| Categoría | Docks/Stands |
| Release date | 15 Jul 2020 |
| Last updated | 05 Nov 2020 |

{: .box-warning}
**Warning**: Restart Required

### 21) ASMedia USB eXtensible Host Controller Driver 

* No lo instalo, es para que los puertos USB se conecten correctamente con los dispositivos detectados

| Fichero | `ASMedia-USB-eXtensible-Host-Controller-Driver_68PNM_WIN_1.16.61.1_A17.EXE` |
| Versión | 1.16.61.1, A17 |
| Categoría | Docks/Stands |
| Release date | 15 Jul 2020 |
| Last updated | 15 Jul 2020 |

{: .box-warning}
**Warning**: Restart Required

## Other

### Intel Thunderbolt Firmware Update

| Fichero | `Intel_TBT3_FW_UPDATE_NVM56_JW7KN_A11_4.56.102.016.exe` |
| Versión | 4.56.102.016, A11 |
| Categoría | Chipset |
| Release date  | 02 Jun 2020  |
| Last updated  | 02 Jun 2020 |

{: .box-warning}
**Warning**: Restart Required

### Dell TPM 2.0 Firmware Update Utility

| Fichero | `DELLTPM_NPCT750_20_V3_7.2.0.2_64.exe` |
| Versión | 7.2.0.2, A03 |
| Categoría | Security |
| Release date | 17 Jul 2020 |
| Last updated | 05 Aug 2020 |

{: .box-warning}
**Warning**: Restart Required

## Exportación

Una vez se ha acabado la instalación de los drivers, éstos se pueden **exportar** para tenerlos disponibles si hubiera que reinstalar el equipo:

```
Export-WindowsDriver -Online -Destination 'D:\EQUIPS\Dell Precision 7530\Export'
```

## PnPutil.exe

Dell explica cómo [instalar los drivers desde un fichero CAB](https://www.dell.com/support/article/es/es/esdhs1/SLN209380/how-to-configure-driver-installation-from-cab-file-s--in-windows--vista--7--8--and-windows-server--2008-2012-?lang=EN), o desde la exportación anterior, usando el comando `pnputil.exe`:

```
mkdir C:\Drivers
expand "D:\EQUIPS\Dell Precision 7530\7530-WIN10-A11-PVTT1.CAB" C:\Drivers -f:*
for /f "tokens=*" %a in ('dir C:\Drivers\*.inf /b /s') do (echo pnputil –i -a "%a")
```

{: .box-warning}
**Warning**: Después hay que esperar un tiempo prudencial para que Windows detecte los dispositivos antes de reiniciar el equipo.

## Driver Pack

Dell dispone de [Driver Packs](https://www.dell.com/support/kbdoc/en-us/000124139/dell-command-deploy-driver-packs-for-enterprise-client-os-deployment) para sus modelos pero los drivers suelen ser más antiguos que los individuales. El driver pack del [Precision 7530](https://www.dell.com/support/kbdoc/en-us/000124115/precision-7530-windows-10-driver-pack) es del 14 Jul 2020 y tiene el identificador PVTT1:

| Fichero | [`7530-WIN10-A11-PVTT1.CAB`](http://downloads.dell.com/FOLDER06292182M/1/7530-WIN10-A11-PVTT1.CAB) |
| Versión | PVTT1 |
| Release date | 14 Jul 2020 |

Estos paquetes de drivers se utilizan con la [Dell Client Command Suite](https://www.delltechnologies.com/en-us/systems-management/client-command-suite.htm) que incluye herramientas para gestionar:

- [Driver Packages](https://www.dell.com/support/article/en-us/sln312414/dell-command-deploy-driver-packs-for-enterprise-client-os-deployment?lang=en)
- [BIOS Management](https://www.dell.com/support/article/en-us/sln311302/dell-command-configure?lang=en) [incluso mediante [Powershell](https://www.dell.com/support/article/en-us/sln311262/dell-command-powershell-provider?lang=en)]
- [Hardware Reporting](https://www.dell.com/support/article/en-us/sln311855/dell-command-monitor?lang=en)
- [Update Management](https://www.dell.com/support/article/en-us/sln311129/dell-command-update?lang=en)
- [Cloud Repository](https://www.dell.com/support/article/sln322893)
- [Out-of-band Management](https://www.dell.com/support/article/en-us/sln311294/dell-command-intel-vpro-out-of-band?lang=en)
- [Workspace One](https://www.delltechnologies.com/en-us/endpointsecurity/manageability.htm)
- [Microsoft Configuration Manager](https://www.dell.com/support/article/en-us/sln311125/dell-command-integration-suite-for-system-center?lang=en) [incluyendo un [catálogo](https://www.dell.com/support/article/en-us/sln311138/dell-command-update-catalog?lang=en) propio]

## Samsung 970 EVO Plus

Después de hacer algunas pruebas de rendimiento del disco [**Samsung 970 EVO Plus**](https://www.samsung.com/semiconductor/minisite/ssd/product/consumer/970evoplus/) he decidido:

* No instalar [Samsung NVM Express Driver 3.3](https://s3.ap-northeast-2.amazonaws.com/global.semi.static/SamsungNVMExpressDriver3.3/7322A6707A720E1A71EF11A3BE1EED819E011D317626415F0281A78151C/Samsung_NVM_Express_Driver_3.3.exe)
* No instalar [Samsung Magician 6.2.1](https://s3.ap-northeast-2.amazonaws.com/global.semi.static/SAMSUNG_SSD_MAGICIAN_201016/SW/5F99E3AA2FB6AC1BAC402B41D766B74675B9E5CD0C0055C39A2A1EB27C8E09EC/Samsung_Magician_Installer.zip)

Mi unidad tiene el firmware **1B2QEXM7** (`Get-PhysicalDisk | Get-StorageFirmwareInformation`) y, aunque existe el **2B2QEXM7**, no lo instalo porque parece que tiene algún [problema de rendimiento](https://www.reddit.com/r/Dell/comments/cfxtoc/samsung_970_evo_plus_performance_in_xps_range/) debido al parámetro `CsEnabled`:

```
reg.exe query "HKLM\System\CurrentControlSet\Control\Power" /v CsEnabled
```

Algunos resultados de las pruebas de rendimiento:

* [UserBenchmark (CsEnabled=1)](https://www.userbenchmark.com/UserRun/24265483)
* [UserBenchMark (CsEnabled=0)](https://www.userbenchmark.com/UserRun/24266251)
* [CrystalDiskMark con Microsoft NVMe Standard Controller](https://twitter.com/manelrodero/status/1218471516532355073)
* [CrystalDiskMark (CsEnabled=1 / CsEnabled=0)](https://twitter.com/manelrodero/status/1223734478859882502)

[1]: /assets/img/laptop-precision-7530.jpg "Foto: dell.com"
