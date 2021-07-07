---
layout : post
blog-width: true
title: 'Añadir drivers a una Offline Standalone Media de ConfigMgr'
date: '2021-02-14 21:44:23'
published: true
tags:
- ConfigMgr
- Drivers
author:
  display_name: Manel Rodero
---

Nuestros compañeros de [UPCnet](https://www.upcnet.es/ca) nos proporcionan una _Standalone Task Sequence Media_ de [ConfigMgr](https://docs.microsoft.com/es-es/mem/configmgr/) para instalar la última versión de Windows 10 del entorno [**GET**](https://serveistic.upc.edu/ca/equip-de-treball).

{: .box-note}
**Nota**: A fecha de hoy, 14 de febrero de 2021, se trata de la versión GET v7 que utiliza Windows 10 1909.

Una [_Standalone Media_](https://www.anoopcnair.com/sccm-configmgr-create-standalone-media/) incluye la secuencia de tareas que automatizan los pasos para instalar el sistema operativo y todo el resto del contenido requerido. Este contenido incluye la imagen de arranque, la imagen del sistema operativo y los controladores de dispositivo.

Aunque esta imagen suele incluir los drivers de red necesarios para que Windows 10 tenga conectividad y pueda completar la unión al dominio, en ocasiones no es así y hay que completar el proceso de forma manual:

* Entrar con la cuenta por defecto del equipo
* Instalar el driver de la tarjeta de red
* Esperar a que Windows detecte el resto de hardware del equipo
* Comprobar el nombre del equipo
* Unir el equipo al dominio
* Reiniciar el equipo para que se apliquen las políticas GPO
* Volver a reiniciarlo para instalar aplicaciones mediante Software Center

{: .box-note}
**Nota**: Los últimos equipos que han tenido este "problema" han sido los **Lenovo ThinkPad L15** y los **HP ProDesk 600 G6**.

Por este motivo hemos decidido comprobar si era posible modificar la imagen de Windows 10 que contiene el fichero `V:\StandaloneTS\GET_UPC_v7.iso`, generado por ConfigMgr, para añadir los drivers de red que faltan.

## Descargar drivers de Lenovo ThinkPad L15

Para descargar los drivers de este equipo lo mejor es buscar por modelo y número de producto en la [web de soporte de Lenovo](https://support.lenovo.com/es/es).

Se accede a la página de soporte del producto [**ThinkPad L15 (type 20U3, 20U4)**](https://pcsupport.lenovo.com/es/es/products/laptops-and-netbooks/thinkpad-l-series-laptops/thinkpad-l15-type-20u3-20u4/) y después a la página para descargar el software y los [controladores](https://pcsupport.lenovo.com/es/es/products/laptops-and-netbooks/thinkpad-l-series-laptops/thinkpad-l15-type-20u3-20u4/downloads/driver-list).

* Redes: LAN (Ethernet) &rarr; Intel PRO/1000 LAN Adapter Software &rarr; [12.18.9.23 (9 septiembre 2020)](https://pcsupport.lenovo.com/es/es/products/laptops-and-netbooks/thinkpad-l-series-laptops/thinkpad-l15-type-20u3-20u4/downloads/ds544116-intel-pro1000-lan-adapter-software-for-windows-10-version-1809-or-later-thinkpad-l14-gen-1-type-20u1-20u2-l15-gen-1-type-20u3-20u4)

Se descarga el fichero [r17rw02w.exe](https://download.lenovo.com/pccbbs/mobiles/r17rw02w.exe) en `V:\StandaloneTS\Downloads` y se procede a su extracción:

```
V:\StandaloneTS\Downloads\r17rw02w.exe /VERYSILENT /DIR=V:\StandaloneTS\Drivers\Lenovo /Extract="YES"
```

El paquete contiene 1 fichero `*.inf`:

```
V:\StandaloneTS\Drivers\Lenovo\e1d68x64.inf
```

El dispositivo que buscamos se encuentra en el fichero `e1d68x64.inf` y corresponde a una tarjeta de red **Intel(R) Ethernet Connection (10) I219-V**.

## Descargar drivers de HP ProDesk 600 G6

Para descargar los drivers de este equipo lo mejor es buscar por modelo y número de producto en la [web de soporte de HP](https://support.hp.com/es-es).

Se accede a la página de soporte del producto [**HP ProDesk 600 G6 MT (9CF30AV)**](https://support.hp.com/es-es/product/hp-prodesk-600-g6-microtower-pc/37884566/model/38461482) y después a la página para descargar el software y los [controladores](https://support.hp.com/es-es/drivers/selfservice/hp-prodesk-600-g6-microtower-pc/37884566/model/38461482).

* Controlador-Red &rarr; Controlador para NIC Intel &rarr; [12.19.0.16 Rev.P (29 diciembre 2020)](https://support.hp.com/es-es/drivers/selfservice/swdetails/hp-prodesk-600-g6-microtower-pc/37884566/model/38461482/swItemId/ob-265942-1)

Se descarga el fichero [sp111991.exe](https://ftp.hp.com/pub/softpaq/sp111501-112000/sp111991.exe) en `V:\StandaloneTS\Downloads` y se procede a su extración:

```
V:\StandaloneTS\Downloads\sp111991.exe /e /f V:\StandaloneTS\Drivers\HP
```

Según el instalador, este paquete contiene drivers para los siguientes modelos de tarjeta:

* Intel Beaver Lake (I210) Desktop Adapter
* Intel Ethernet Connection I219-LM
* Intel Ethernet Connection I219-V
* Intel Ethernet Connection I225-LM
* Intel Ethernet Connection I225-V

En nuestro caso necesitamos el driver para el dispositivo `PCI\VEN_8086&DEV_0D4C`. El paquete contiene 3 ficheros `*.inf` diferentes:

```
V:\StandaloneTS\Drivers\HP\src\Drivers\E1D\e1d68x64.inf
V:\StandaloneTS\Drivers\HP\src\Drivers\E1R\e1r68x64.inf
V:\StandaloneTS\Drivers\HP\src\Drivers\E2F\e2f68.inf
```

El dispositivo que buscamos se encuentra en el fichero `e1d68x64.inf` y corresponde a una tarjeta de red **Intel(R) Ethernet Connection (11) I219-LM**.

## Extraer el contenido del fichero ISO

Se utiliza **7-Zip** para extraer el contenido del fichero `GET_UPC_v7.iso` en el directorio `V:\StandaloneTS\ISO`.

## Montar la imagen de Windows 10

Para poder inyectar los drivers, es necesario montar la imagen de Windows 10 utilizada por la _standalone task sequence_. Se puede observar que no existe un fichero `*.wim` sino que la imagen está dividida en ficheros `*.swm` más pequeños:

```
V:\StandaloneTS\ISO\SMS\PKG\UPC00134\6D63CE6489090BD2AE3632E341D41772C9EC90DDC2BDC0E61BCC62822D7A3A96000.swm
V:\StandaloneTS\ISO\SMS\PKG\UPC00134\6D63CE6489090BD2AE3632E341D41772C9EC90DDC2BDC0E61BCC62822D7A3A96001.swm
V:\StandaloneTS\ISO\SMS\PKG\UPC00134\6D63CE6489090BD2AE3632E341D41772C9EC90DDC2BDC0E61BCC62822D7A3A96002.swm
```

{: .box-note}
**Nota**: Los ficheros `*.swm` corresponden a una [imagen `*.wim` dividida](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/split-a-windows-image--wim--file-to-span-across-multiple-dvds) para que se pueda grabar en múltiples DVDs.

Una imagen WIM dividida en ficheros SWM no se puede modificar por lo que es necesario [juntar los ficheros `*.swm`](https://www.techmazza.com/convert-export-swim-wim/).

Se puede obtener información sobre la imagen ejecutando los siguientes comandos desde un CMD elevado:

```
mkdir V:\StandaloneTS\temp
xcopy.exe V:\StandaloneTS\ISO\SMS\PKG\UPC00134\*.swm V:\StandaloneTS\temp
dism.exe /get-wiminfo /wimfile:V:\StandaloneTS\temp\6D63CE6489090BD2AE3632E341D41772C9EC90DDC2BDC0E61BCC62822D7A3A96000.swm
```

```
Deployment Image Servicing and Management tool
Version: 10.0.19041.1

Details for image : V:\StandaloneTS\temp\6D63CE6489090BD2AE3632E341D41772C9EC90DDC2BDC0E61BCC62822D7A3A96000.swm

Index : 1
Name : W10-C-BAS64-07
Description : <undefined>
Size : 29,543,351,666 bytes

The operation completed successfully.
```

A continuación se genera el fichero `*.wim` mediante el siguiente comando:

```
cd V:\StandaloneTS\temp
dism.exe /export-image /sourceimagefile:6D63CE6489090BD2AE3632E341D41772C9EC90DDC2BDC0E61BCC62822D7A3A96000.swm /swmfile:"6D63CE6489090BD2AE3632E341D41772C9EC90DDC2BDC0E61BCC62822D7A3A9600*.swm" /sourceindex:1 /destinationimagefile:6D63CE6489090BD2AE3632E341D41772C9EC90DDC2BDC0E61BCC62822D7A3A96000.wim
```

```
Deployment Image Servicing and Management tool
Version: 10.0.19041.1

Exporting image
[==========================100.0%==========================]
The operation completed successfully.
```

Este fichero `*.wim` se puede montar con el siguiente comando:

```
mkdir V:\StandaloneTS\mount
dism.exe /mount-image /imagefile:6D63CE6489090BD2AE3632E341D41772C9EC90DDC2BDC0E61BCC62822D7A3A96000.wim /index:1 /mountdir:V:\StandaloneTS\mount
```
```
Deployment Image Servicing and Management tool
Version: 10.0.19041.1

Mounting image
[==========================100.0%==========================]
The operation completed successfully.
```

## Inyectar los drivers

Finalmente, se puede inyectar los drivers que sean necesarios:

* Lenovo ThinkPad L15

```
dism.exe /image:V:\StandaloneTS\mount /add-driver /driver:V:\StandaloneTS\Drivers\Lenovo\e1d68x64.inf
```
```
Deployment Image Servicing and Management tool
Version: 10.0.19041.1

Image Version: 10.0.18363.900

Found 1 driver package(s) to install.
Installing 1 of 1 - V:\StandaloneTS\Drivers\Lenovo\e1d68x64.inf: The driver package was successfully installed.
The operation completed successfully.
```

<p></p>

* HP ProDesk 600 G6

```
dism.exe /image:V:\StandaloneTS\mount /add-driver /driver:V:\StandaloneTS\Drivers\HP\src\Drivers\E1D\e1d68x64.inf
```
```
Deployment Image Servicing and Management tool
Version: 10.0.19041.1

Image Version: 10.0.18363.900

Found 1 driver package(s) to install.
Installing 1 of 1 - V:\StandaloneTS\Drivers\HP\src\Drivers\E1D\e1d68x64.inf: The driver package was successfully installed.
The operation completed successfully.
```

## Desmontar la imagen de Windows 10

A continuación se pueden aplicar los cambios usando el modificar `/commit` (si se quisieran descartar se usaría el `/discard`):

```
dism.exe /unmount-image /mountdir:V:\StandaloneTS\mount /commit
```
```
Deployment Image Servicing and Management tool
Version: 10.0.19041.1

Saving image
[==========================100.0%==========================]
Unmounting image
[==========================100.0%==========================]
The operation completed successfully.
```

## Generar los ficheros `*.swm`

Finalmente, hay que volver a dividir la imagen `*.wim` en ficheros `*.swm`:

{: .box-note}
**Nota**: No sabemos si sería posible dejar la imagen `*.wim` en lugar de los ficheros `*.swm` porque la _task sequence_ está ofuscada.

```
cd V:\StandaloneTS\temp
del *.swm
dism.exe /split-image /imagefile:6D63CE6489090BD2AE3632E341D41772C9EC90DDC2BDC0E61BCC62822D7A3A96000.wim /swmfile:6D63CE6489090BD2AE3632E341D41772C9EC90DDC2BDC0E61BCC62822D7A3A96000.swm /filesize:4096
```
```
Deployment Image Servicing and Management tool
Version: 10.0.19041.1

The operation completed successfully.
```

{: .box-note}
**Nota**: Es necesario renombrar los ficheros porque `dism.exe` los deja como `~000.swm`, `~0002.swm` y `~0003.swm`. Los originales se llamaban `~000.swm`, `~001.swm` y `~002.swm`.

```
ren 6D63CE6489090BD2AE3632E341D41772C9EC90DDC2BDC0E61BCC62822D7A3A960002.swm 6D63CE6489090BD2AE3632E341D41772C9EC90DDC2BDC0E61BCC62822D7A3A96001.swm
ren 6D63CE6489090BD2AE3632E341D41772C9EC90DDC2BDC0E61BCC62822D7A3A960003.swm 6D63CE6489090BD2AE3632E341D41772C9EC90DDC2BDC0E61BCC62822D7A3A96002.swm
```

Se puede comprobar la información del nuevo fichero `*.swm`:

```
dism.exe /get-wiminfo /wimfile:6D63CE6489090BD2AE3632E341D41772C9EC90DDC2BDC0E61BCC62822D7A3A96000.swm
```
```
Deployment Image Servicing and Management tool
Version: 10.0.19041.1

Details for image : 6D63CE6489090BD2AE3632E341D41772C9EC90DDC2BDC0E61BCC62822D7A3A96000.swm

Index : 1
Name : W10-C-BAS64-07
Description : <undefined>
Size : 29,512,263,344 bytes

The operation completed successfully.
```

## Sustituir los ficheros originales

Finalmente se pueden sustituir los ficheros originales del ISO por los que se acaban de generar:

```
xcopy.exe /y *.swm V:\StandaloneTS\ISO\SMS\PKG\UPC00134
```

## Generar un nuevo fichero ISO

Se puede utilizar el comando `oscdimg.exe` para crear un fichero ISO que funcione correctamente en BIOS y en UEFI.

{: .box-note}
**Nota**: Para realizarlo se necesitan los ficheros `etfsboot.com` y `efisys.bin` que se encuentran en cualquier ISO de Windows 10 (p.ej. los ficheros de una ISO de Windows 10 Education x64 20H2).

```
xcopy.exe "V:\OSDBuilder\Share\OSImport\Windows 10 Education x64 2009 19042.508\OS\boot\etfsboot.com" V:\StandaloneTS\ISO\boot
xcopy.exe "V:\OSDBuilder\Share\OSImport\Windows 10 Education x64 2009 19042.508\OS\efi\microsoft\boot\efisys.bin" V:\StandaloneTS\ISO\EFI\Microsoft\Boot
"C:\Program Files (x86)\Windows Kits\10\Assessment and Deployment Kit\Deployment Tools\amd64\Oscdimg\oscdimg.exe" -m -o -u2 -udfver102 -bootdata:2#p0,e,bV:\StandaloneTS\ISO\boot\etfsboot.com#pEF,e,bV:\StandaloneTS\ISO\EFI\Microsoft\Boot\efisys.bin V:\StandaloneTS\ISO V:\StandaloneTS\GET_UPC_v7_Mod.iso
```
```
OSCDIMG 2.56 CD-ROM and DVD-ROM Premastering Utility
Copyright (C) Microsoft, 1993-2012. All rights reserved.
Licensed only for producing Microsoft authorized content.


Scanning source tree
Scanning source tree complete (383 files in 164 directories)

Computing directory information complete

Image file is 12390498304 bytes (before optimization)

Writing 383 files in 164 directories to V:\StandaloneTS\GET_UPC_v7_Mod.iso

100% complete

Storage optimization saved 59 files, 21141504 bytes (1% of image)

After optimization, image file is 12370575360 bytes
Space saved because of embedding, sparseness or optimization = 21141504

Done.
```
<p></p>
