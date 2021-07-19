---
layout : post
blog-width: true
title: 'Instalar Samba en Raspberry Pi'
date: '2021-02-28 18:59:31'
published: true
tags:
- Raspberry
author:
  display_name: Manel Rodero
---

La instalación de [OpenMediaVault 5](https://www.openmediavault.org/) en una Raspberry Pi 4 no funciona todo lo bien que esperaba y he decidido instalar [Samba](https://www.samba.org/) para compartir ficheros con los ordenadores Windows.

En la [documentación de Raspberry.org](https://www.raspberrypi.org/documentation/remote-access/samba.md) se indican los comandos necesarios para instalar:

```
# Actualizar repositorios
sudo apt update

# Instalar servidor Samba
sudo apt install samba samba-common-bin -y

# (Opcional) Instalar cliente Samba
sudo apt install smbclient cifs-utils -y
```

Durante la instalación de los paquetes aparece el siguiente mensaje al cual hay que contestar `No` ya que no se utiliza WINS en esta instalación:

```
If your computer gets IP address information from a DHCP server on the network, the DHCP server may
also provide information about WINS servers ("NetBIOS name servers") present on the network.

This requires a change to your smb.conf file so that DHCP-provided WINS settings will automatically
be read from /var/lib/samba/dhcp.conf.

The dhcp-client package must be installed to take advantage of this feature.

Modify smb.conf to use WINS settings from DHCP?
```

## Standalone Server

Se realizará la configuración de Samba como un [Standalone Server](https://wiki.samba.org/index.php/Setting_up_Samba_as_a_Standalone_Server) con el nivel de seguridad de tipo **Usuario**.

Se paran los servicios mientras se realiza esta configuración:

```
sudo service nmbd stop
sudo service smbd stop
```

Se realiza una copia de seguridad del fichero original:

```
cd /etc/samba
sudo mv smb.conf smb.conf.orig
```

A continuación se crea un nuevo fichero `/etc/samba/smb.conf` con el siguiente contenido:

```
[global]
   server string = samba.home
   server role = standalone server
   interfaces = lo eth0
   bind interfaces only = yes
   disable netbios = yes
   smb ports = 445
   log file = /var/log/samba/smb.log
   max log size = 10000
   security = user
```

## Añadir disco para datos

A continuación se detalla cómo particionar y formatear el disco `/dev/sdb` que se utilizará para almacenar los datos de este pequeño NAS.

Primero se instala la utilidad `gdisk` que permite particionar correctamente en formato **GPT**:

```
sudo apt install gdisk -y
```

A continuación se crea una partición que ocupa todo el disco (1TB):

```
sudo gdisk /dev/sdb
```
```
GPT fdisk (gdisk) version 1.0.3

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries.

Command (? for help): o
This option deletes all partitions and creates a new protective MBR.
Proceed? (Y/N): y

Command (? for help): n
Partition number (1-128, default 1):
First sector (34-1953525134, default = 2048) or {+-}size{KMGTP}:
Last sector (2048-1953525134, default = 1953525134) or {+-}size{KMGTP}:
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300):
Changed type of partition to 'Linux filesystem'

Command (? for help): p
Disk /dev/sdb: 1953525168 sectors, 931.5 GiB
Model: USB3.0
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 673344AA-2122-4F13-9131-15B6B926E126
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 1953525134
Partitions will be aligned on 2048-sector boundaries
Total free space is 2014 sectors (1007.0 KiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048      1953525134   931.5 GiB   8300  Linux filesystem

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sdb.
The operation has completed successfully.

```

Se formatea la partición `/dev/sdb1` usando el sistema de ficheros Linux **ext4**:

```
sudo mkfs.ext4 -L data /dev/sdb1
```
```
mke2fs 1.44.5 (15-Dec-2018)
Creating filesystem with 244190385 4k blocks and 61054976 inodes
Filesystem UUID: 331658d1-b085-4830-92d3-5d5b72a4f620
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968,
        102400000, 214990848

Allocating group tables: done
Writing inode tables: done
Creating journal (262144 blocks): done
Writing superblocks and filesystem accounting information: done
```

Se crea un directorio para montar la partición:

```
sudo mkdir /data
sudo mount /dev/sdb1 /data
```

Y se añade la siguiente línea al fichero `/etc/fstab` para que se monte siempre que se reinicie el sistema operativo:

```
UUID=331658d1-b085-4830-92d3-5d5b72a4f620 /data ext4 defaults 0 0
```

{: .box-note}
**Nota**: El **UUID** se ha obtenido obtener usando los comandos `df -hT`, `lsblk -o name,label,UUID /dev/sdb` y `blkid`.

## Carpeta para Samba

Se crea una carpeta donde estarán el resto de carpetas compartidas mediante Samba:

```
sudo mkdir /data/homes
sudo chown :sambashare /data/homes
```

## Creación de usuario

Se crea el usuario que accederá a las carpetas compartidas, asignando la prote   cción [2770](https://chmodcommand.com/chmod-2770/) que incluye [`setgid`](https://linuxconfig.org/how-to-use-special-permissions-the-setuid-setgid-and-sticky-bits):

```
# Crear home directory:
sudo mkdir /data/homes/manel

# Usuario sin home directory y sin acceso shell
sudo adduser --home /data/homes/manel --no-create-home --shell /usr/sbin/nologin --ingroup sambashare manel

# Cambiar las protecciones de los directorios
sudo chown manel:sambashare /data/homes/manel
sudo chmod 2770 /data/homes/manel

# Añadir el usuario a la BD de Samba y habilitarlo
sudo smbpasswd -a manel
sudo smbpasswd -e manel

# Añadir el usuario al grupo SambaShare
sudo usermod -aG sambashare $(whoami)
sudo usermod -aG sambashare manel
```

## Carpetas compartidas

Y se añade la siguiente definición al fichero `/etc/samba/smb.conf`:

```
[manel]
   path = /data/samba/manel
   browseable = no
   read only = no
   force create mode = 0660
   force directory mode = 2770
   valid users = manel
```

Si se quisiera crear una carpeta compartida para cualquiera que pertenezca al grupo `sambashare` se podría hacer de la siguiente manera:

```
[everyone]
   path = /data/samba/everyone
   browseable = no
   read only = no
   force create mode = 0660
   force directory mode = 2770
   valid users = @sambashare
```

## Reiniciar Samba

Finalmente, es necesario reiniciar el _daemon_ de Samba para que se apliquen los cambios:

```
sudo service smbd restart
```

{: .box-note}
**Nota**: Al no utilizar NetBIOS en este entorno, se puede deshabilitar el _daemon_ [`nmbd`](https://www.samba.org/samba/docs/current/man-html/nmbd.8.html) mediante el siguiente comando: `sudo systemctl disable nmbd`.

## Acceder desde Windows

Y después se podría mapear una unidad desde el Explorador de Archivos (`Computer > Map Network Drive`) o desde la línea de comandos:

```
net use z: \\192.168.1.180\manel /user:manel
```

## Referencias

* [How To Set Up a Samba Share For A Small Organization on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-samba-share-for-a-small-organization-on-ubuntu-16-04)