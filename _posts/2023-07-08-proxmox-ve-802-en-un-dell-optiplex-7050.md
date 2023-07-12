---
layout : post
blog-width: true
title: 'Proxmox VE 8.0.2 en un Dell OptiPlex 7050'
date: '2023-07-08 19:17:06'
published: true
tags:
- HomeLab
author:
  display_name: Manel Rodero
---

# Introducción

En este artículo explicaré como he instalado [Proxmox VE](https://www.proxmox.com/en/proxmox-ve) en un Mini PC **OptiPlex 7050 Micro** [que compré hace unos días](mini-pc-dell-optiplex-7050-micro) con la idea de complementar (o sustituir) a la [Raspberry Pi 4](instalar-raspberry-pi-os-64bits) que estoy usando como _Home Server_ actualmente.

# Proxmox VE 8.0.2

**Proxmox VE** es una plataforma de virtualización de código abierto que integra un hipervisor KVM, contenedores de Linux (LXC) y almacenamiento definido por software (SDS).

La [instalación](https://pve.proxmox.com/pve-docs/chapter-pve-installation.html) es bastante sencilla utilizando la [ISO](https://www.proxmox.com/en/downloads/item/proxmox-ve-8-0-iso-installer) que se descarga desde su web.

> **Nota**: Usaré un _pendrive_ con [Ventoy](https://www.ventoy.net/en/index.html) en el que he copiado el fichero `proxmox-ve_8.0-2.iso` ya que ésto evita tener que escribir la imagen usando herramientas como [Etcher](https://etcher.balena.io/) o [Rufus](https://rufus.ie/en/).

* Iniciar el ordenador y pulsar `F12` para acceder al menú de arranque
* Seleccionar el _pendrive_ para iniciarlo mediante UEFI
* Seleccionar la ISO de Proxmox en el menú de Ventoy
* Seleccionar `Boot in normal mode` en Ventoy
* Seleccionar `Install Proxmox VE (Graphical)` en el asistente de Proxmox
* Aceptar la licencia pulsando el botón `I agree`
* Seleccionar las siguientes opciones de instalación:
  * Proxmox Virtual Environment (PVE)
    * Target Harddisk: `/dev/sdb (238,47GiB, Micron 1100 SATA)`
    * Filesytem: `ext4`
    * hdsize: **214.0** (para dejar espacio de [_over provisioning_](https://www.minitool.com/partition-disk/ssd-over-provisioning.html))
  * Location and Time Zone selection
    * Country: **Spain**
    * Time zone: **Europe/Madrid**
    * Keyboard Layout: `Spanish`
  * Administration Password and Email Address
    * Password: xxxxxxxx
    * Confirm: xxxxxxxx
  * Management Network Configuration
    * Management interface: `enp0s31f6 (e1000e)`
    * Hostname (FQDN): `pve.home`
    * IP Address (CIDR): **192.168.1.190/24**
    * Gateway: `192.168.1.1`
    * DNS Server: `192.168.1.1`
* Marcar la opción para reiniciar automáticamente después de la instalación
* Pulsar el botón `Install`

## Configuración

Una vez instalado, se puede acceder a la interfaz de gestión en [`https://192.168.1.190:8006`](https://192.168.1.190:8006) y autenticarse mediante el usuario `root` y la contraseña indicada anteriormente.

> **Nota**: Como estamos usando la versión gratuita, sin una suscripción válida, siempre aparecerá un mensaje indicándolo y no podremos actualizar Debian usando los repositorios de tipo **Enterprise**.

Es posible utilizar unos repositorios alternativos para actualizar Debian haciendo los siguientes cambios desde una _shell_ (`Datacenter` &rarr; `pve` &rarr; `Shell`):

```
cd /etc/apt/sources.list.d
cp pve-enterprise.list pve-no-subscription.list

# Deshabilitar los repositorios que necesitan suscripción (añadiendo un # al principio)
sed -i 's/^/#/' pve-enterprise.list
sed -i 's/^/#/' ceph.list

# Definir el repositorio que no necesita suscripción
# Cambiar https://enterprise por http://download y pve-enterprise por pve-no-subscription
sed -i 's/https:\/\/enterprise/http:\/\/download/g;s/pve-enterprise/pve-no-subscription/g' pve-no-subscription.list
```

En la página de repositorios (`Datacenter` &rarr; `pve` &rarr; `Updates` &rarr; `Repositories`) aparecerá un mensaje indicando que ahora existe un repositorio correcto para actualizar (`pve-no-subscription`) pero que éste no está recomendado en entornos de producción ya que puede contener paquetes que no son estables.

A continuación se actualizan las fuentes mediante el comando `apt-get update`:

```
root@pve:~# apt-get update
Get:1 http://security.debian.org bookworm-security InRelease [48.0 kB]
Get:2 http://security.debian.org bookworm-security/main amd64 Packages [47.3 kB]
Get:3 http://security.debian.org bookworm-security/main Translation-en [25.9 kB]
[...]
Get:7 http://download.proxmox.com/debian/pve bookworm InRelease [2,768 B]
Get:8 http://download.proxmox.com/debian/pve bookworm/pve-no-subscription amd64 Packages [78.6 kB]
Get:9 http://ftp.es.debian.org/debian bookworm/main Translation-en [6,076 kB]
[...]
Fetched 15.5 MB in 2s (6,764 kB/s)
Reading package lists... Done
```

Y, finalmente, se ejecuta un [`apt-get upgrade`](https://itsfoss.com/apt-get-upgrade-vs-dist-upgrade/) o un [`apt-get dist-upgrade`](https://itsfoss.com/apt-get-upgrade-vs-dist-upgrade/) para actualizar los paquetes:

```
root@pve:~# apt-get upgrade
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Calculating upgrade... Done
The following packages will be upgraded:
  bind9-dnsutils bind9-host bind9-libs ifupdown2 libgstreamer-plugins-base1.0-0 libpve-cluster-api-perl libpve-cluster-perl libpve-common-perl
  libpve-http-server-perl libpve-storage-perl proxmox-backup-client proxmox-backup-file-restore proxmox-mail-forward proxmox-widget-toolkit pve-cluster
  pve-container pve-docs pve-i18n
18 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
Need to get 24.4 MB of archives.
After this operation, 7,988 kB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Get:1 http://security.debian.org bookworm-security/main amd64 bind9-host amd64 1:9.18.16-1~deb12u1 [301 kB]
[...]
Get:16 http://download.proxmox.com/debian/pve bookworm/pve-no-subscription amd64 pve-container all 5.0.4 [133 kB]
[...]
Fetched 24.4 MB in 4s (6,965 kB/s)
Reading changelogs... Done
(Reading database ... 45081 files and directories currently installed.)
Preparing to unpack .../00-bind9-host_1%3a9.18.16-1~deb12u1_amd64.deb ...
[...]
Preparing to unpack .../15-pve-container_5.0.4_all.deb ...
Unpacking pve-container (5.0.4) over (5.0.3) ...
[...]
Setting up proxmox-backup-file-restore (3.0.1-1) ...
Updating file-restore initramfs...
12101 blocks
[...]
Setting up bind9-host (1:9.18.16-1~deb12u1) ...
[...]
Setting up pve-container (5.0.4) ...
Processing triggers for pve-manager (8.0.3) ...
Processing triggers for man-db (2.11.2-2) ...
Processing triggers for pve-ha-manager (4.0.2) ...
Processing triggers for libc-bin (2.36-9) ...
```

## Añadir _storage_

La instalación predeterminada de Proxmox particiona el disco en 2 partes:

* `local`: utilizado para almacenar imágenes ISO, plantillas de contenedores, etc.
* `local-lvm`: imágenes de disco y contenedores

Pero, ¿cómo se puede agregar almacenamiento adicional? (por ejemplo, el segundo disco SSD SATA que he añadido al ordenador).

Primero hay que crear una partición con todo el espacio del disco (reservando un 10% para OP), formatearla con `ext4` y montarla en un directorio (en mi caso utilizaré el directorio `/data`):

* Abrir una _Shell_
* Ejecutar el comando `lsblk`
* Tomar nota del disco que se quiere añadir (en este caso se trata del `sda` de 931.5G)
* Usar `fdisk /dev/sda` para [crear una partición](https://www.digitalocean.com/community/tutorials/create-a-partition-in-linux):
  * Escribir `g` para crear una tabla de particiones GPT
  * Escribir `n` para crear una nueva partición de **838.4G** (se deja un 10% para OP)
  * Escribir `t` para cambiar el tipo de partición a **Linux LVM** usando el valor **43**
  * Escribir `p` para ver cómo ha queda la partición
  * Escribir `w` para guardar los cambios
* Usar `mkfs.ext4 -L data /dev/sda1` para formatear la partición y etiquetarla como **data**
* Usar `mkdir -p /data` para crear un punto de montaje para esta partición
* Usar `blkid /dev/sda1` para comprobar el **UUID** que se utilizará en el fichero `/etc/fstab` (también se podría utilizar **LABEL**)
* Editar el fichero `/etc/fstab` y añadir la entrada correspondiente a la partición:

```
UUID="5faab86f-d389-4a7c-896e-977d2fd6e3cb" /data ext4 defaults 0 2
```

* Usar `mount -a` para montar las particiones
* Usar `systemctl daemon-reload ` para recargar los cambios en `systemd`

A continuación, ya se puede añadir el directorio `/data` en Proxmox:

* Abrir `Datacenter` &rarr; `Storage` &rarr; `Add` &rarr; `Directory`)
  * ID = **data**
  * Directory = **/data**
  * Content = seleccionar todas las opciones (Disk image, ISO image, Container template, VZDump backup file, Container, Snippets)
* Pulsar el botón `Add` para añadir

Los siguientes pasos serán comenzar a "jugar" con los contenedores LXC y Docker, pero eso lo explicaré en los siguientes _posts_ ;-)

# Referencias

* [How to add storage drive on Proxmox](https://serverdecode.com/add-drive-proxmox/)
* [How to Add Additional Storage On Proxmox?](https://sysadminote.com/how-to-add-additional-storage-on-proxmox/)
