---
layout : post
blog-width: true
title: 'Reemplazo del disco de sistema de Proxmox'
date: '2024-11-26 18:05:50'
last-updated: '2025-01-01 20:14:50'
published: true
tags:
- Proxmox
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2024-11-26_cover.png"
thumbnail-img: ""
---

Creo que estoy llegando al límite de capacidad en el disco de sistema del [Proxmox VE que instalé en un Dell OptiPlex 7050](proxmox-ve-802-en-un-dell-optiplex-7050) hace algo más de un año, en Julio de 2023.

Por este motivo he decidido investigar un poco cómo funciona el **almacenamiento en Proxmox** para ver si es necesario reemplazarlo por otro de mayor capacidad.

# Discos SSD

Inicialmente se hizo una instalación básica de Proxmox VE en el disco **`/dev/sdb`** (Micron 1100 SATA 256GB) de **238.47GiB** usando **ext4** como sistema de ficheros y un `hdsize` de **214.0** para dejar un 10% de espacio de _over provisioning_.

Se puede utilizar el comando `smartctl -a /dev/sdb` para obtener información sobre este disco:

```
Model Family:     Crucial/Micron Client SSDs
Device Model:     Micron 1100 SATA 256GB
Serial Number:    <Redactado>
LU WWN Device Id: 5 00a075 114d65a16
Firmware Version: M0DL002
User Capacity:    256,060,514,304 bytes [256 GB]
Sector Sizes:     512 bytes logical, 4096 bytes physical
Rotation Rate:    Solid State Device
Form Factor:      M.2
TRIM Command:     Available, deterministic, zeroed
Device is:        In smartctl database 7.3/5319
ATA Version is:   ACS-3 T13/2161-D revision 5
SATA Version is:  SATA 3.2, 6.0 Gb/s (current: 6.0 Gb/s)
Local Time is:    Tue Nov 26 18:17:14 2024 CET
SMART support is: Available - device has SMART capability.
SMART support is: Enabled
```

Al finalizar la instalación se añadió un segundo disco **`/dev/sda`** (Crucial CT1000MX500SSD1) de **931.51GiB** usando **ext4** como sistema de ficheros y un `hdsize` de **838.4** para dejar también un 10% de OP.

La información sobre este segundo disco se obtiene usando el comando `smartctl -a /dev/sda`:

```
Model Family:     Crucial/Micron Client SSDs
Device Model:     CT1000MX500SSD1
Serial Number:    <Redactado>
LU WWN Device Id: 5 00a075 1e69da803
Firmware Version: M3CR045
User Capacity:    1,000,204,886,016 bytes [1.00 TB]
Sector Sizes:     512 bytes logical, 4096 bytes physical
Rotation Rate:    Solid State Device
Form Factor:      2.5 inches
TRIM Command:     Available
Device is:        In smartctl database 7.3/5319
ATA Version is:   ACS-3 T13/2161-D revision 5
SATA Version is:  SATA 3.3, 6.0 Gb/s (current: 6.0 Gb/s)
Local Time is:    Tue Nov 26 18:22:57 2024 CET
SMART support is: Available - device has SMART capability.
SMART support is: Enabled
```

# Logical Volume Manager (LVM)

Proxmox VE utiliza por defecto el _framework_ [**Logical Volume Manager** (LVM)](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)){:target=_blank} para gestionar el disco local seleccionado durante el proceso de instalación (en mi caso `/dev/sdb`).

* Un **physical volume** (PV) está formados por un único disco o partición física
* Un **volume group** (VG) está formado por uno o más PVs
* Los **logical volumes** (LVs), una especie de "particiones virtuales", se crean dentro de los VGs

![LVM (Wikimedia)][1]

## Particiones

Se puede utilizar el comando `fdisk -l` para mostrar información sobre las particiones creadas en cada uno de los discos instalados en el servidor Proxmox:

```
root@pve:~# fdisk -l /dev/sdb
Disk /dev/sdb: 238.47 GiB, 256060514304 bytes, 500118192 sectors
Disk model: Micron 1100 SATA
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: 7BF8C68E-6BD5-4ADA-99E7-76A2AF97B415

Device       Start       End   Sectors  Size Type
/dev/sdb1       34      2047      2014 1007K BIOS boot
/dev/sdb2     2048   2099199   2097152    1G EFI System
/dev/sdb3  2099200 448790528 446691329  213G Linux LVM

Partition 1 does not start on physical sector boundary.
```
```
root@pve:~# fdisk -l /dev/sda
Disk /dev/sda: 931.51 GiB, 1000204886016 bytes, 1953525168 sectors
Disk model: CT1000MX500SSD1 
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: 2432B13C-852B-2344-9022-06AC8CA99F67

Device     Start        End    Sectors   Size Type
/dev/sda1   2048 1758220287 1758218240 838.4G Linux LVM
```

{: .box-note}
Se observa que las particiones `/dev/sdb3` y `/dev/sda1` son de tipo **Linux LVM**, el tipo utilizado por defecto en Proxmox.

El comando `partx -s` también permite obtener información sobre las particiones de los discos:

```
root@pve:~# partx -s /dev/sdb
NR   START       END   SECTORS  SIZE NAME UUID
 1      34      2047      2014 1007K      2f60d165-bb81-430c-bc83-8c25ba48ffef
 2    2048   2099199   2097152    1G      e7ee1e90-f3f1-40aa-a0a6-b4afe1ffe08b
 3 2099200 448790528 446691329  213G      191002ca-91c8-409d-9987-9905b56884b3
```
```
root@pve:~# partx -s /dev/sda
NR START        END    SECTORS   SIZE NAME UUID
 1  2048 1758220287 1758218240 838.4G      9bc0bc79-7737-1b41-802d-27db38d0151d
```

## Physical Volume

En mi caso, durante la instalación inicial, se creó un **physical volume (PV)** en **`/dev/sdb3`**, tal como muestra la salida del comando `pvs -v`:

```
root@pve:~# pvs -v
  PV         VG  Fmt  Attr PSize    PFree  DevSize  PV UUID                               
  /dev/sdb3  pve lvm2 a--  <213.00g 16.00g <213.00g yXMM4U-NI1V-d2Bx-LGZq-zdXV-BxRD-VuDisz
```

Se puede utilizar el comando `pvdisplay` para mostrar información sobre este PV:

```
root@pve:~# pvdisplay
  --- Physical volume ---
  PV Name               /dev/sdb3
  VG Name               pve
  PV Size               <213.00 GiB / not usable 3.00 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              54527
  Free PE               4097
  Allocated PE          50430
  PV UUID               yXMM4U-NI1V-d2Bx-LGZq-zdXV-BxRD-VuDisz
```

{: .box-note}
El PV `/dev/sdb3`, que forma parte del VG **`pve`**, tiene un tamaño de <213.00 GiB de los cuales quedan libres 16.00 GiB (4097 PE de 4.00 MiB). Un PE o **physical extent** es la unidad más pequeñas en la que se divide un PV.

## Volume Group

Dentro del PV `/dev/sdb3`se creó un único **volume group (VG)** llamado **`pve`**, tal como muestra la salida del comando `vgs -v`:

```
root@pve:~# vgs -v
  VG  Attr   Ext   #PV #LV #SN VSize    VFree  VG UUID                                VProfile
  pve wz--n- 4.00m   1  21   0 <213.00g 16.00g sFUh8l-RtdT-YCZA-vTJb-6YET-EUM9-2HTwU6
```

Se puede utilizar el comando `vgdisplay` para mostrar información sobre este VG:

```
root@pve:~# vgdisplay
  --- Volume group ---
  VG Name               pve
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  12361
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                21
  Open LV               17
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <213.00 GiB
  PE Size               4.00 MiB
  Total PE              54527
  Alloc PE / Size       50430 / 196.99 GiB
  Free  PE / Size       4097 / 16.00 GiB
  VG UUID               sFUh8l-RtdT-YCZA-vTJb-6YET-EUM9-2HTwU6
```

{: .box-note}
El VG `pve` tiene un tamaño de <213.00 GiB de los cuales quedan libres 16.00 GiB (4097 PE de 4.00 MiB). Un PE o **physical extent** es la unidad más pequeñas en la que se divide un PV.

## Logical Volumes

Finalmente, dentro del VG `pve` se asignaron los **logical volumes (LV)** **`data`**, **`root`** y **`swap`**, tal como muestra la salida del comando `lvs`:

```
root@pve:~# lvs
  LV               VG  Attr       LSize    Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  data             pve twi-aotz-- <123.24g             47.45  2.80                            
  root             pve -wi-ao----  <63.25g                                                    
  swap             pve -wi-ao----    8.00g
```

{: .box-note}
El LV `data`, que utiliza **Thin Provisioning** tal como indica el atributo `twi-aotz--`, tiene un **tamaño virtual** de <123.24g pero únicamente el 47.45% de este espacio está efectivamente utilizado.

Se puede utilizar el comando `lvdisplay` para mostrar información sobre estos LVs:

```
root@pve:~# lvdisplay 
  --- Logical volume ---
  LV Name                data
  VG Name                pve
  LV UUID                hOJQqQ-IlAz-YMu6-JuDf-LK7H-sXZ5-cosYyZ
  LV Write Access        read/write (activated read only)
  LV Creation host, time proxmox, 2023-07-09 13:27:59 +0200
  LV Pool metadata       data_tmeta
  LV Pool data           data_tdata
  LV Status              available
  # open                 0
  LV Size                <123.24 GiB
  Allocated pool data    47.44%
  Allocated metadata     2.79%
  Current LE             31549
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:5
   
  --- Logical volume ---
  LV Path                /dev/pve/swap
  LV Name                swap
  VG Name                pve
  LV UUID                s5m0EI-15w1-ZtIM-Puep-zYbi-SUs5-xL9mTv
  LV Write Access        read/write
  LV Creation host, time proxmox, 2023-07-09 13:27:55 +0200
  LV Status              available
  # open                 2
  LV Size                8.00 GiB
  Current LE             2048
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:0
   
  --- Logical volume ---
  LV Path                /dev/pve/root
  LV Name                root
  VG Name                pve
  LV UUID                jFldLS-iJn5-YO2j-63X5-VYn7-1OSD-saekeD
  LV Write Access        read/write
  LV Creation host, time proxmox, 2023-07-09 13:27:56 +0200
  LV Status              available
  # open                 1
  LV Size                <63.25 GiB
  Current LE             16191
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:1
```

El contenido de cada uno de estos LVs es el siguiente:

* El LV `root` (`/dev/pve/root`) contiene el sistema operativo Proxmox montado en `/`
* El LV `swap` (`/dev/pve/swap`) se utiliza como espacio de _swap_
* El LV `data` contiene las imágenes de disco de las máquinas virtuales

{: .box-note}
El LV `data` no tiene un **LV Path** porque se trata de un **LV-thin pool**. Los datos se almacenan directamente en el _pool_ y se gestionan a través de las imágenes de disco.

# Tipos de _storage_

Los diferentes tipos de **_storage_** que se pueden utilizar en Proxmox se clasifican en dos clases:

* **File**: sistema tradicional que permite acceder a un sistema de ficheros de tipo POSIX. Aquí se puede almacenar todo tipo de contenido
* **Block**: permite almacenar imágenes _raw_ grandes y soportan _snapshots_ y _clones_. No es posible guardar otro tipo de ficheros (ISO, _backups_, etc.)

En mi instalación, al ser bastante simple, únicamente se utilizan los siguientes tipos de almacenamiento:

| Descripción | Plugin type | Level | Shared | Snapshots | Stable |
| -- | -- | -- | -- | -- | -- |
| Directory | `dir` | file | no | no (sí, usando `qcow2`) | yes |
| LVM | `lvm` | block | no (sí, usándolo sobre iSCSI) | no | yes |
| LVM-thin | `lvmthin` | block | no | yes | yes |

{: .box-note}
El [capítulo 7 de la "Proxmox VE Administration Guide"](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#chapter_storage){:target=_blank} está dedicado completamente al almacenamiento en Proxmox VE.

La configuración de los _storage pools_ de Proxmox se define desde `Datacenter` &rarr; `Storage`. En mi caso tengo los siguientes a día de hoy:

![Storage Pools][2]

La configuración de estos _storage pools_ se guarda en el fichero `/etc/pve/storage.cfg`:

```
root@pve:~# cat /etc/pve/storage.cfg
dir: local
        path /var/lib/vz
        content iso,vztmpl
        shared 0

# default image store on LVM based installation
lvmthin: local-lvm
        thinpool data
        vgname pve
        content rootdir,images

dir: datos
        path /datos
        content backup
        prune-backups keep-all=1
        shared 0
```

Los diferentes contenidos que se puede almacenar en los _storage pool_ son:

* `images`: Imágenes de máquinas virtuales QEMU/KVM
* `rootdir`: Datos de los contenedores LXC
* `vztmpl`: Plantillas de contenedores LXC
* `backup`: Ficheros de _backup_ (`vzdump`)
* `iso`: Imágenes ISO
* `snippets`: Fragmentos de fichero, por ejemplo _guest hook scripts_

El comando `pvesm status` permite obtener información sobre el estado actual de los _storage pools_:

```
root@pve:~# pvesm status
Name             Type     Status           Total            Used       Available        %
datos             dir     active       864183016       476347896       343863280   55.12%
local             dir     active        64962140         7360388        54269452   11.33%
local-lvm     lvmthin     active       129224704        61368811        67855892   47.49%
```

Se puede ver el contenido de un _storage pool_ usando el comando `pvesm list <storage_pool>` tal como se muestra a continuación:

```
root@pve:~# pvesm list datos
Volid                                                    Format  Type            Size VMID
datos:backup/vzdump-lxc-300-2024_11_21-03_00_45.tar.zst  tar.zst backup     329562204 300
[...]
datos:backup/vzdump-qemu-101-2024_11_27-03_00_05.vma.zst vma.zst backup    1829078542 101
```
```
oot@pve:~# pvesm list local
Volid                                                    Format  Type           Size VMID
local:vztmpl/debian-11-turnkey-core_17.1-1_amd64.tar.gz  tgz     vztmpl    206782882
local:vztmpl/debian-12-turnkey-core_18.0-1_amd64.tar.gz  tgz     vztmpl    211493902
local:vztmpl/debian-12-standard_12.2-1_amd64.tar.zst     tzst    vztmpl    126129049
local:vztmpl/debian-12-standard_12.7-1_amd64.tar.zst     tzst    vztmpl    126515062
local:vztmpl/ubuntu-22.04-standard_22.04-1_amd64.tar.zst tzst    vztmpl    129824858
```
```
root@pve:~# pvesm list local-lvm 
Volid                      Format  Type             Size VMID
local-lvm:base-700-disk-0  raw     images    17179869184 700
local-lvm:vm-101-disk-0    raw     images        4194304 101
local-lvm:vm-101-disk-1    raw     images    34359738368 101
local-lvm:vm-300-disk-0    raw     rootdir    4294967296 300
local-lvm:vm-301-disk-0    raw     rootdir    2147483648 301
[...]
local-lvm:vm-313-disk-0    raw     rootdir    4294967296 313
local-lvm:vm-700-cloudinit raw     images        4194304 700
```

## _Storage pool_ `local-lvm` vs _Volume group_ `pve`

Entonces, ¿qué relación hay entre el _storage pool_ `local-lvm` y el _volume group_ `pve`?

* **Volume Group (VG)** `pve`: Durante la instalación de Proxmox se crea un VG llamado `pve`. Este VG se utiliza para gestionar el espacio de almacenamiento del sistema
* **Logical Volume (LV)** `data`: Dentro del VG `pve`, se crean varios LVs. Uno de ellos, llamado `data`, es un **LV-thin pool**. Este LV se utiliza para almacenar imágenes de máquinas virtuales (VMs) y snapshots de manera eficiente
* **Storage Pool** `local-lvm`: El storage pool `local-lvm` hace referencia a este LV `data`. Es un _pool_ de almacenamiento LVM-thin que permite la creación de snapshots y clones de manera eficiente

![local-lvm][3]

# ¿Necesito un disco más grande?

Podemos ejecutar el comando `lvs -a` para mostrar todos los volúmenes lógicos, incluso los componentes internos y los segmentos que normalmente no se muestran:

```
root@pve:~# lvs -a
  LV               VG  Attr       LSize    Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  base-700-disk-0  pve Vri---tz-k   16.00g data                                               
  data             pve twi-aotz-- <123.24g             47.49  2.80                            
  [data_tdata]     pve Twi-ao---- <123.24g                                                    
  [data_tmeta]     pve ewi-ao----    1.25g                                                    
  [lvol0_pmspare]  pve ewi-------    1.25g                                                    
  root             pve -wi-ao----  <63.25g                                                    
  swap             pve -wi-ao----    8.00g                                                    
  vm-101-disk-0    pve Vwi-aotz--    4.00m data        0.00                                   
  vm-101-disk-1    pve Vwi-aotz--   32.00g data        16.53                                  
  vm-300-disk-0    pve Vwi-aotz--    4.00g data        98.63                                  
  vm-301-disk-0    pve Vwi-aotz--    2.00g data        99.51                                  
  vm-302-disk-0    pve Vwi-aotz--   20.00g data        72.66                                  
  vm-303-disk-0    pve Vwi-aotz--    2.00g data        99.77                                  
  vm-304-disk-0    pve Vwi-aotz--    8.00g data        60.37                                  
  vm-305-disk-0    pve Vwi-aotz--    8.00g data        36.88                                  
  vm-306-disk-0    pve Vwi-aotz--    4.00g data        99.83                                  
  vm-307-disk-0    pve Vwi-a-tz--   11.00g data        25.58                                  
  vm-308-disk-0    pve Vwi-aotz--    2.00g data        56.48                                  
  vm-309-disk-0    pve Vwi-aotz--    4.00g data        56.38                                  
  vm-310-disk-0    pve Vwi-a-tz--    8.00g data        97.73                                  
  vm-311-disk-0    pve Vwi-aotz--    4.00g data        46.16                                  
  vm-312-disk-0    pve Vwi-aotz--    4.00g data        32.45                                  
  vm-313-disk-0    pve Vwi-aotz--    4.00g data        32.27                                  
  vm-700-cloudinit pve Vwi-a-tz--    4.00m data        9.38
```

La suma del tamaño asignado a cada uno de los volúmenes lógicos correspondientes a los discos de las máquinas virtuales y de los contenedores LXC es de **135GB**. Si el porcentaje de uso de estos discos fuese del 100% se superaría los **123,24GB** asignados al _storage pool_ `data`.

Ahora bien, hay algo que no cuadra y no consigo entender el porqué. Por ejemplo, si examino el LV `vm-300-disk-0` usando el comando `lvdisplay` se muestra que el **98.63%** de los **4.00 GiB** está ocupado:

```
root@pve:~# lvdisplay /dev/pve/vm-300-disk-0
  --- Logical volume ---
  LV Path                /dev/pve/vm-300-disk-0
  LV Name                vm-300-disk-0
  VG Name                pve
  LV UUID                Cf3Gji-VJgm-3YkL-nj2P-AlAz-Vtnc-jcLUV9
  LV Write Access        read/write
  LV Creation host, time pve, 2023-11-01 20:59:45 +0100
  LV Pool name           data
  LV Status              available
  # open                 1
  LV Size                4.00 GiB
  Mapped size            98.63%
  Current LE             1024
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:6
```

En cambio, si accedemos al contenedor LXC 300 y ejecutamos el comando `df -kh` se muestra únicamente un **31%** de uso:

```
root@samba ~# df -kh
Filesystem                        Size  Used Avail Use% Mounted on
/dev/mapper/pve-vm--300--disk--0  3.9G  1.2G  2.6G  31% /
none                              492K  4.0K  488K   1% /dev
udev                              7.7G     0  7.7G   0% /dev/tty
tmpfs                             7.7G     0  7.7G   0% /dev/shm
tmpfs                             3.1G  1.5M  3.1G   1% /run
tmpfs                             5.0M     0  5.0M   0% /run/lock
```

¿Alguien sabría explicarme el motivo?

## Errores al crear LXC

En la situación explicada anteriormente, la creación de un LXC con un tamaño de disco de **8GB** genera el siguiente error:

```
WARNING: You have not turned on protection against thin pools running out of space.
WARNING: Set activation/thin_pool_autoextend_threshold below 100 to trigger automatic extension of thin pools before they get full.
Logical volume "vm-314-disk-0" created.
WARNING: Sum of all thin volume sizes (<141.01 GiB) exceeds the size of thin pool pve/data and the amount of free space in volume group (16.00 GiB).
```

Queda claro que, aunque no me cuadren los números, al crear este nuevo LXC estaría por encima de la capacidad real disponible en el disco y eso podría provocar corrupción o pérdida de datos.

# Actualización del disco

Para poder actualizar el disco de sistema he comprado un nuevo **disco M.2 NVMe** y una **carcasa** compatible:

* [WD Blue SN580 1 TB, M.2 NVMe SSD, PCIe Gen4 x4](https://amzn.to/4iWCFMt){:target=_blank}, Amazon, 56,99€
* [UGREEN Carcasa M.2 NVMe PCIe USB C 3.2 Gen 2](https://amzn.to/4gGIeNv){:target=_blank}, Amazon, 22,49€

La idea es la siguiente:

* Colocar el nuevo disco M.2 NVMe en la carcasa y conectarlo al servidor usando el puerto USB-C del frontal
* Acceder a la _shell_ de Proxmox y comprobar que el nuevo disco es detectado
* Reiniciar el servidor y arrancar desde una ISO de Clonezilla
* Clonar el disco de sistema en el nuevo disco
* Apagar el servidor, desconectar la carcasa y sacar el nuevo disco clonado
* Intercambiar el disco de sistema por el nuevo disco clonado
* Poner en marcha el servidor y comprobar que Proxmox se inicia correctamente
* Reiniciar el servidor y arrancar desde una ISO de GParted
* Ampliar la partición de sistema para que ocupe el 90% del nuevo disco
* Reiniciar el servidor y comprobar que Proxmox se inicia correctamente
* Extender el volúmen lógico `data`
* Comprobar que se pueden crear LXC/VM sin errores

## Nuevo disco `/dev/sdc`

Una vez conectada la carcasa al puerto USB-C del frontal, el comando `fdisk -l` muestra que se ha detectado un nuevo disco `/dev/sdc`.

La salida del comando anterior muestra los dos discos físicos existentes (`/dev/sda` y `/dev/sdb`):

```
root@pve:~# fdisk -l
Disk /dev/sda: 931.51 GiB, 1000204886016 bytes, 1953525168 sectors
Disk model: CT1000MX500SSD1 
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: 2432B13C-852B-2344-9022-06AC8CA99F67

Device     Start        End    Sectors   Size Type
/dev/sda1   2048 1758220287 1758218240 838.4G Linux LVM


Disk /dev/sdb: 238.47 GiB, 256060514304 bytes, 500118192 sectors
Disk model: Micron 1100 SATA
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: 7BF8C68E-6BD5-4ADA-99E7-76A2AF97B415

Device       Start       End   Sectors  Size Type
/dev/sdb1       34      2047      2014 1007K BIOS boot
/dev/sdb2     2048   2099199   2097152    1G EFI System
/dev/sdb3  2099200 448790528 446691329  213G Linux LVM

Partition 1 does not start on physical sector boundary.


Disk /dev/sdc: 931.51 GiB, 1000204886016 bytes, 1953525168 sectors
Disk model: SN580 1TB       
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
```

{: .box-note}
El comando `fdisk -l` también muestra los volúmenes lógicos `/dev/mapper/xxxxx` correspondientes a los discos de los LXC/VM mapeados a través de LVM, pero se han omitido de la salida anterior.

## Clonezilla

Para realizar el **clonado** del disco `/dev/sdb` en el disco `/dev/sdc` he utilizado [Clonezilla](https://clonezilla.org/){:target=_blank}, una herramienta gratuita y de código abierto que permite clonar y hacer imágenes de discos.

He descargado la versión **stable 3.2.0-5** de Clonezilla en formato `ISO` para `amd64` y la he grabado en un _pendrive_ que tengo formateado con [Ventoy](https://www.ventoy.net/en/index.html){:target=_blank}.

A continuación he reiniciado el servidor y he pulsado `F12` para seleccionar el _pendrive_ anterior y arrancar desde él. Una vez dentro de Ventoy se ha seleccionado el fichero `clonezilla-live-3.2.0-5-amd64.iso` para iniciar Clonezilla:

* Seleccionar Clonezilla live (VGA 800x600)
* Lenguaje: English
* Keyboard layout: US keyboard
* Ejecutar Clonezilla
* Seleccionar **device-device**
* Seleccionar Beginner mode
* Seleccionar **disk_to_local_disk**
* Seleccionar source: `sdb 256GB_Micron_1100_SATA_pci`
* Seleccionar target: `sdc 1000GB_SN580_1TB__pci`
* Seleccionar `-sfsck` (_skip checking/repairing source file system_)
* Seleccionar `-k0` (_use the partition table from the source disk_)
* Seleccionar `-p` choose (_choose reboot/shutdown/etc when everything is finished_)

{: .box-note}
Según se puede ver en los mensajes que aparecen en pantalla, las opciones seleccionadas equivalen al siguiente comando: `/usr/sbin/ocs-onthefly -g auto -e1 auto -e2 -r -j2 -sfsck -k0 -p choose -f sdb -d sdc`.

Clonezilla va realizando diferentes procesos y mostrando la salida por pantalla (con algunos _warning_ o errores que se supone no tienen importancia) y, al final, se ejecutan diferentes comandos relacionados con LVM  (`lvresize`, `resize2fs`, `e2fsck`, etc.)

Una vez finalizado el proceso, elijo la opción _**poweroff**_ para apagar el servidor e intercambiar los discos NVMe.

## En Proxmox por primera vez

Una vez reemplazado el disco NVMe, se pone en marcha el servidor y se comprueba que Proxmox se inicia correctamente. El disco NVMe se detecta en `/dev/nvme0n1`.

Se puede utilizar el comando `smartctl -a /dev/nvme0n1` para obtener información sobre este disco:

```
smartctl 7.3 2022-02-28 r5338 [x86_64-linux-6.8.12-2-pve] (local build)
Copyright (C) 2002-22, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Model Number:                       WD Blue SN580 1TB
Serial Number:                      <Redactado>
Firmware Version:                   281010WD
PCI Vendor/Subsystem ID:            0x15b7
IEEE OUI Identifier:                0x001b44
Total NVM Capacity:                 1,000,204,886,016 [1.00 TB]
Unallocated NVM Capacity:           0
Controller ID:                      0
NVMe Version:                       1.4
Number of Namespaces:               1
Namespace 1 Size/Capacity:          1,000,204,886,016 [1.00 TB]
Namespace 1 Formatted LBA Size:     512
Namespace 1 IEEE EUI-64:            001b44 4a41d2b9f0
Local Time is:                      Wed Jan  1 12:25:22 2025 CET
Firmware Updates (0x14):            2 Slots, no Reset required
Optional Admin Commands (0x0017):   Security Format Frmw_DL Self_Test
Optional NVM Commands (0x00df):     Comp Wr_Unc DS_Mngmt Wr_Zero Sav/Sel_Feat Timestmp Verify
Log Page Attributes (0x7e):         Cmd_Eff_Lg Ext_Get_Lg Telmtry_Lg Pers_Ev_Lg *Other*
Maximum Data Transfer Size:         256 Pages
Warning  Comp. Temp. Threshold:     84 Celsius
Critical Comp. Temp. Threshold:     88 Celsius
Namespace 1 Features (0x02):        NA_Fields

Supported Power States
St Op     Max   Active     Idle   RL RT WL WT  Ent_Lat  Ex_Lat
 0 +     4.80W    4.80W       -    0  0  0  0        0       0
 1 +     3.30W    3.00W       -    0  0  0  0        0       0
 2 +     2.20W    2.00W       -    0  0  0  0        0       0
 3 -   0.0150W       -        -    3  3  3  3     1500    2500
 4 -   0.0050W       -        -    4  4  4  4    10000    6000
 5 -   0.0033W       -        -    5  5  5  5   176000   25000

Supported LBA Sizes (NSID 0x1)
Id Fmt  Data  Metadt  Rel_Perf
 0 +     512       0         2
 1 -    4096       0         1

=== START OF SMART DATA SECTION ===
SMART overall-health self-assessment test result: PASSED

SMART/Health Information (NVMe Log 0x02)
Critical Warning:                   0x00
Temperature:                        43 Celsius
Available Spare:                    100%
Available Spare Threshold:          10%
Percentage Used:                    0%
Data Units Read:                    10,659 [5.45 GB]
Data Units Written:                 447,368 [229 GB]
Host Read Commands:                 234,195
Host Write Commands:                1,774,791
Controller Busy Time:               2
Power Cycles:                       12
Power On Hours:                     1
Unsafe Shutdowns:                   2
Media and Data Integrity Errors:    0
Error Information Log Entries:      0
Warning  Comp. Temperature Time:    0
Critical Comp. Temperature Time:    0
Temperature Sensor 1:               64 Celsius
Temperature Sensor 2:               40 Celsius

Error Information (NVMe Log 0x01, 16 of 256 entries)
No Errors Logged
```

El comando `fdisk -l` muestra que las particiones de este disco coinciden exactamente (tamaño, inicio, final, etc.) con las que tenía el disco original, es decir la **clonación** ha sido un éxito:

```
root@pve:~# fdisk -l /dev/nvme0n1
Disk /dev/nvme0n1: 931.51 GiB, 1000204886016 bytes, 1953525168 sectors
Disk model: WD Blue SN580 1TB                       
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 7BF8C68E-6BD5-4ADA-99E7-76A2AF97B415

Device           Start       End   Sectors  Size Type
/dev/nvme0n1p1      34      2047      2014 1007K BIOS boot
/dev/nvme0n1p2    2048   2099199   2097152    1G EFI System
/dev/nvme0n1p3 2099200 448790528 446691329  213G Linux LVM
```

La ejecución de los comandos de Proxmox para interactuar con los PV, VG y LV muestran los mismos datos que obtuve inicialmente cuando comencé a mirar este tema.

Physical volumes (PV):

```
root@pve:~# pvs -v
  PV             VG  Fmt  Attr PSize    PFree  DevSize  PV UUID                               
  /dev/nvme0n1p3 pve lvm2 a--  <213.00g 16.00g <213.00g yXMM4U-NI1V-d2Bx-LGZq-zdXV-BxRD-VuDisz
```

```
root@pve:~# pvdisplay 
  --- Physical volume ---
  PV Name               /dev/nvme0n1p3
  VG Name               pve
  PV Size               <213.00 GiB / not usable 3.00 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              54527
  Free PE               4097
  Allocated PE          50430
  PV UUID               yXMM4U-NI1V-d2Bx-LGZq-zdXV-BxRD-VuDisz
```

Volume groups (VG):

```
root@pve:~# vgs -v
  VG  Attr   Ext   #PV #LV #SN VSize    VFree  VG UUID                                VProfile
  pve wz--n- 4.00m   1  21   0 <213.00g 16.00g sFUh8l-RtdT-YCZA-vTJb-6YET-EUM9-2HTwU6 
```

```
root@pve:~# vgdisplay
  --- Volume group ---
  VG Name               pve
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  14161
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                21
  Open LV               17
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <213.00 GiB
  PE Size               4.00 MiB
  Total PE              54527
  Alloc PE / Size       50430 / 196.99 GiB
  Free  PE / Size       4097 / 16.00 GiB
  VG UUID               sFUh8l-RtdT-YCZA-vTJb-6YET-EUM9-2HTwU6
```

Logical volumes (LV):

```
root@pve:~# lvs
  LV               VG  Attr       LSize    Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  base-700-disk-0  pve Vri---tz-k   16.00g data                                               
  data             pve twi-aotz-- <123.24g             47.98  2.82                            
  root             pve -wi-ao----  <63.25g                                                    
  swap             pve -wi-ao----    8.00g   
```

```
root@pve:~# lvdisplay 
  --- Logical volume ---
  LV Name                data
  VG Name                pve
  LV UUID                hOJQqQ-IlAz-YMu6-JuDf-LK7H-sXZ5-cosYyZ
  LV Write Access        read/write (activated read only)
  LV Creation host, time proxmox, 2023-07-09 13:27:59 +0200
  LV Pool metadata       data_tmeta
  LV Pool data           data_tdata
  LV Status              available
  # open                 0
  LV Size                <123.24 GiB
  Allocated pool data    47.98%
  Allocated metadata     2.82%
  Current LE             31549
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:5
   
  --- Logical volume ---
  LV Path                /dev/pve/swap
  LV Name                swap
  VG Name                pve
  LV UUID                s5m0EI-15w1-ZtIM-Puep-zYbi-SUs5-xL9mTv
  LV Write Access        read/write
  LV Creation host, time proxmox, 2023-07-09 13:27:55 +0200
  LV Status              available
  # open                 2
  LV Size                8.00 GiB
  Current LE             2048
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:0
   
  --- Logical volume ---
  LV Path                /dev/pve/root
  LV Name                root
  VG Name                pve
  LV UUID                jFldLS-iJn5-YO2j-63X5-VYn7-1OSD-saekeD
  LV Write Access        read/write
  LV Creation host, time proxmox, 2023-07-09 13:27:56 +0200
  LV Status              available
  # open                 1
  LV Size                <63.25 GiB
  Current LE             16191
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:1
```

Una vez comprobado que todo funciona igual que con el disco original, llega el momento de utilizar todo el espacio existente en el nuevo disco.

## GParted

Para realizar la **ampliación** de la partición `/dev/nvme0n1p3` he utilizado [GParted](https://gparted.org/){:target=_blank}, una herramienta gratuita que permite gestionar las particiones.

He descargado la versión **1.6.0-10** de GParted en formato `ISO` para `amd64` y la he grabado en un _pendrive_ que tengo formateado con [Ventoy](https://www.ventoy.net/en/index.html){:target=_blank}.

A continuación he reiniciado el servidor y he pulsado `F12` para seleccionar el _pendrive_ anterior y arrancar desde él. Una vez dentro de Ventoy se ha seleccionado el fichero `gparted-live-1.6.0-10-amd64.iso` para iniciar GParted:

* Seleccionar Gparted live (VGA 800x600)
* Keymap: Don't touch keymap
* Language: 33 US English
* Ejecutar GParted
* Seleccionar `/dev/nvme0n1p3 213.00GiB; Usados 197.00GiB (92%); Sin usar 16.00GiB (8%)`
* Usar las teclas `ALT+P` para seleccionar el menú "Partition"
* Seleccionar "Resize/Move" y usar los siguientes valores:
  * New size: **857560MiB**
  * Free space following (MiB) **95284** (dejo un 10% libre para OP igual que estaba el disco original)
* Usar el tabulador para pulsar el botón "Resize"
* Usar el tabulador para pulsar el botón "Apply"
* Aceptar el mensaje para aplicar las operaciones pendientes
* Usar el tabulador para pulsar el botón "Close"
* Usar las teclas `ALT+G` para seleccionar el menú "GParted"
* Seleccionar "Quit"
* Usar las teclas `ALT+F1` para abrir una terminal
* Ejecutar el comando `sudo shutdown -h now`
* Quitar el _pendrive_ y pulsar ENTER para apagar el servidor

{: .box-note}
GParted ejecuta un comando similar al siguiente `lvm pvresize -v --yes --setphysicalvolumesize 878141440K '/dev/nvme0n1p3'` para realizar esta ampliación.

## En Proxmox por segunda vez

Una vez ampliada la partición, se pone en marcha el servidor y se comprueba que Proxmox sigue iniciando correctamente.

El comando `fdisk -l` muestra que la partición se ha ampliado:

```
root@pve:~# fdisk -l /dev/nvme0n1
Disk /dev/nvme0n1: 931.51 GiB, 1000204886016 bytes, 1953525168 sectors
Disk model: WD Blue SN580 1TB                       
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 7BF8C68E-6BD5-4ADA-99E7-76A2AF97B415

Device           Start        End    Sectors   Size Type
/dev/nvme0n1p1      34       2047       2014  1007K BIOS boot
/dev/nvme0n1p2    2048    2099199    2097152     1G EFI System
/dev/nvme0n1p3 2099200 1758382079 1756282880 837.5G Linux LVM
```

Además, vemos como GParted ha ampliado el _physical volume_ (PV) correspondiente:

```
root@pve:~# pvs -v
  PV             VG  Fmt  Attr PSize    PFree   DevSize PV UUID                               
  /dev/nvme0n1p3 pve lvm2 a--  <837.46g 540.46g 837.46g yXMM4U-NI1V-d2Bx-LGZq-zdXV-BxRD-VuDisz
```

```
root@pve:~# pvdisplay 
  --- Physical volume ---
  PV Name               /dev/nvme0n1p3
  VG Name               pve
  PV Size               <837.46 GiB / not usable 3.00 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              214389
  Free PE               138359
  Allocated PE          76030
  PV UUID               yXMM4U-NI1V-d2Bx-LGZq-zdXV-BxRD-VuDisz
```

Y también se ha expandido el _volume group_ (VG):

```
root@pve:~# vgs -v
  VG  Attr   Ext   #PV #LV #SN VSize    VFree   VG UUID                                VProfile
  pve wz--n- 4.00m   1  22   0 <837.46g 540.46g sFUh8l-RtdT-YCZA-vTJb-6YET-EUM9-2HTwU6
```

```
root@pve:~# vgdisplay 
  --- Volume group ---
  VG Name               pve
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  14378
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                22
  Open LV               17
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <837.46 GiB
  PE Size               4.00 MiB
  Total PE              214389
  Alloc PE / Size       76030 / 296.99 GiB
  Free  PE / Size       138359 / 540.46 GiB
  VG UUID               sFUh8l-RtdT-YCZA-vTJb-6YET-EUM9-2HTwU6
```

{: .box-note}
GParted, en versiones recientes, expande automáticamente el PV al nuevo tamaño de la partición. LVM, que es muy eficiente en la gestión de volúmenes, detecta automáticamente el espacio adicional en el PV y ajusta el VG para incluir este nuevo espacio.

## Extender el _storage pool_

Una vez ampliado el tamaño de la **partición** &rarr; PV &rarr; VG, se procede a ampliar el tamaño del LV `pve/data` (_storage pool_ donde se guardan los discos de las VM y LXC).

Para ello se utiliza el comando `lvextend` y, en este caso, se realiza una ampliación de 100GB:

```
root@pve:~# lvextend -L +100G pve/data
  Size of logical volume pve/data_tdata changed from <123.24 GiB (31549 extents) to <223.24 GiB (57149 extents).
  Logical volume pve/data successfully resized.
```

```
root@pve:~# pvesm status
Name             Type     Status           Total            Used       Available        %
datos             dir     active       864183016       503012928       317198248   58.21%
local             dir     active        64962140         7442572        54187268   11.46%
local-lvm     lvmthin     active       234082304        62640424       171441879   26.76%
```

{: .box-note}
Se podría haber usado `+100%FREE` para ocupar todo el espacio disponible en el VG.

# Micron 1100 SATA

El Micron 1100 SATA es un disco M.2 PCIe NVMe de 3ª generación, Clase 40 y tamaño 2280 (22m x 80mm).

El servidor Dell OptiPlex 7050 Micro tiene un _socket_ M.2 que soporta tanto discos SATA M.2 como discos **PCI NVMe (PCI 3.0 x4)**. La siguiente tabla muestra algunos discos compatibles y no excesivamente caros:

| **Disco** | **Capacidad** | **Velocidad de lectura secuencial** | **Velocidad de escritura secuencial** | **Interfaz** | **Tecnología NAND** | **Factor de forma** | **Garantía** |
|-----------|---------------|------------------------------------|--------------------------------------|--------------|---------------------|---------------------|--------------|
| **Micron 1100** | 275 GB - 2 TB | Hasta 530 MB/s | Hasta 500 MB/s | SATA | TLC | 2.5" y M.2 | 3 años |
| [**Crucial P3 1TB M.2 PCIe Gen3 NVMe SSD**](https://amzn.to/4gYUfxq){:target=_blank} | 1 TB | 3500 MB/s | 3000 MB/s | PCIe 3.0 | QLC | M.2 (2280) | 5 años |
| [**Crucial P3 Plus SSD 1TB PCIe Gen4 NVMe M.2**](https://amzn.to/40kZguE){:target=_blank} | 1 TB | 5000 MB/s | 4200 MB/s | PCIe 4.0 | QLC | M.2 (2280) | 5 años |
| [**WD Blue SN580 1 TB, M.2 NVMe SSD, PCIe Gen4 x4**](https://amzn.to/3DTV2kT){:target=_blank} | 1 TB | 5600 MB/s | 3100 MB/s | PCIe 4.0 | TLC | M.2 (2280) | 5 años |
| [**Samsung 980 1 TB PCIe 3.0**](https://amzn.to/3DSmorz){:target=_blank} | 1 TB | 3500 MB/s | 3300 MB/s | PCIe 3.0 | TLC | M.2 (2280) | 5 años |

# Referencias

* [Proxmox VE Administration Guide](https://pve.proxmox.com/pve-docs/pve-admin-guide.pdf){:target=_blank}
* [Brief Introduction to Logical Volume Manager (LVM) – Concept and example of application](https://www.brainupdaters.net/es/brief-introduction-logical-volumes-lv-concept-example-application-2/){:target=_blank}
* [The Complete Beginner's Guide to LVM in Linux](https://linuxhandbook.com/lvm-guide/){:target=_blank}
* [Linux Logical Volume Manager (LVM) tutorial](https://linuxconfig.org/linux-lvm-logical-volume-manager){:target=_blank}
* [Dell OptiPlex 7050M](https://www.hardware-corner.net/desktop-models/Dell-OptiPlex-7050M/){:target=_blank}
* [What is M.2? Keys and Sockets Explained](https://www.atpinc.com/blog/what-is-m.2-M-B-BM-key-socket-3){:target=_blank}
* [M.2 M vs M.2 (B+M): What’s the Difference?](https://www.partitionwizard.com/partitionmanager/m2-m-vs-bm.html){:target=_blank}

### Historial de cambios

* **2024-11-26**: Documento inicial (investigación y datos técnicos)
* **2025-01-01**: Clonezilla/GParted

[1]: /assets/img/blog/2024-11-26_image_1.jpg "LVM (Wikimedia)"
[2]: /assets/img/blog/2024-11-26_image_2.jpg "Storage Pools"
[3]: /assets/img/blog/2024-11-26_image_3.jpg "local-lvm"
