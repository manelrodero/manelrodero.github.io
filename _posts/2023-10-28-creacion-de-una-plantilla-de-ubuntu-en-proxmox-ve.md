---
layout : post
blog-width: true
title: 'Creación de una plantilla de Ubuntu en Proxmox VE'
date: '2023-10-28 17:16:18'
published: true
tags:
- Proxmox
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2023-10-28_cover.png"
thumbnail-img: ""  
---

# Introducción

Una de las acciones más repetitivas que se realizan en [Proxmox VE](https://www.proxmox.com/en/proxmox-virtual-environment/overview){:target="_blank"} es la [creación de **máquinas virtuales**](https://pve.proxmox.com/pve-docs/chapter-qm.html#qm_virtual_machines_settings){:target="_blank"}. Cada vez que se crea una VM hay que definir sus parámetros generales, el sistema operativo que se quiere instalar, el tamaño y tipo del disco que tendrán, el número de CPU y memoria RAM que se asignará, la dirección IP, etc.

La [creación de una **plantilla**](https://pve.proxmox.com/pve-docs/chapter-qm.html#qm_templates){:target="_blank"} permite simplificar este proceso preparando una máquina virtual con unas determinadas características que después se marcará como _read only_ y se podrá [**clonar**](https://pve.proxmox.com/pve-docs/chapter-qm.html#qm_copy_and_clone){:target="_blank"} tantas veces como se desee.

Para crear esta plantilla se utilizará una [imagen oficial de **Ubuntu Cloud**](https://cloud-images.ubuntu.com/){:target="_blank"}, concretamente una [**Ubuntu Minimal 22.04 LTS**](https://cloud-images.ubuntu.com/minimal/releases/jammy/release/){:target="_blank"} (_Jammy Jellyfish_), que se desplegará utilizando [**cloud-init**](https://cloudinit.readthedocs.io/en/latest/){:target="_blank"} para inicializar el usuario y la configuración de red.

{: .box-note}
**Nota**: La creación de una plantilla se puede realizar tanto desde la interfaz gráfica como desde la _shell_. Pero, como se verá a continuación, algunas veces es necesario realizar acciones en ambas.

# A. Usando la interfaz gráfica

## A1. Crear una máquina virtual

Dado que una plantilla es tipo especial de máquina virtual, lo primero que hay que hacer es crearla, indicando las características mínimas que se requieren en el entorno en que se utilizará.

* Expandir el centro de datos (_Datacenter_)
* Seleccionar el servidor (_node_): `pve`
* Pulsar con el botón derecho y seleccionar la opción **Create VM**
  * **General**
    * Node: pve
    * VM ID: `700` (se usará este rango para plantillas)
    * Name: `ubuntu-2204-template`
  * **OS**
    * Escoger la opción `Do not use any media` (ya que no se usará ninguna ISO)
    * Type: Linux
    * Version: 6.x - 2.6 Kernel
  * **System**
    * Graphic card: Default
    * Machine: Default (i440fx)
    * BIOS: Default (SeaBIOS)
    * SCSI Controller: VirtIO SCSI single
    * Habilitar la opción `Qemu Agent`
  * **Disks**
    * Borrar el disco `scsi0` (ya que se usará una _cloud image_ descargada desde Ubuntu)
  * **CPU**
    * Sockets: 1
    * Cores: 1
    * Type: x86-64-v2-AES
  * **Memory**
    * Memory (MiB): `1024`
  * **Network**
    * Bridge: vmbr0
    * VLAN Tag: no VLAN
    * Firewall: Habilitado
    * Model: VirtIO (paravirtualized)
    * MAC address: auto
  * **Confirm**  
    * No habilitar la opción `Start after created`
* Pulsar el botón **Finish**
* Esperar a que finalice la creación de la VM

## A2. Añadir una unidad de **cloud-init**

Proxmox VE utiliza una unidad de tipo CDROM para pasar los datos de [Cloud-Init](https://cloudinit.readthedocs.io/en/latest/){:target="_blank"} a la máquina virtual. Por tanto es necesario añadir una unidad de este tipo a la máquina virtual.

* Expandir el centro de datos (_Datacenter_)
* Expandir el servidor (_node_): `pve`
* Seleccionar la máquina virtual: `700 (ubuntu-2204-template)`
* Seleccionar la opción **Hardware**
* Pulsar el botón **Add**
* Selecciónar la opción `CloudInit Drive`
  * Escoger el storage `local-lvm` (obliga a usar _Raw disk image_)
* Pulsar el botón **Add**

## A3. Configurar las opciones de **cloud-init**

A continuación se configuran los valores por defecto que tendrá la máquina virtual (por ejemplo, el usuario/contraseña o la clave SSH para permitir la conexión a la misma después de un despliegue:

* Expandir el centro de datos (_Datacenter_)
* Expandir el servidor (_node_): `pve`
* Seleccionar la máquina virtual: `700 (ubuntu-2204-template)`
* Selecciónar la opción **Cloud-Init**
* Editar los siguientes valores:
  * Username: `manel`
  * Password: `P@ssw0rd!` (no se recomienda utilizar contraseña y mucho menos la indicada; es mejor usar una clave pública SSH)
  * SSH public key: `manel` (cargar la clave desde un fichero)
  * IP Config (net0): `DHCP` (para IPv4)

{: .box-note}
**Nota**: Se puede [generar una clave SSH](autenticacion-ssh-usando-una-clave-privada) usando el comando `ssh-keygen -t ecdsa -b 521 -f id_ecdsa -C "manel"` y después añadir la clave pública a la plantilla.

## A4. Regenerar la imagen de **cloud-init**

Después de cambiar los valores en el punto anterior, es necesario regenerar la imagen **ISO** que se utilizará para pasarlos a la máquina virtual durante el inicio de la misma.

* Expandir el centro de datos (_Datacenter_)
* Expandir el servidor (_node_): `pve`
* Seleccionar la máquina virtual: `700 (ubuntu-2204-template)`
* Selecciónar la opción **Cloud-Init**
* Pulsar el botón **Regenerate image**

## A5. Configurar la consola de la VM

En muchas imágenes Cloud-Init es necesario configurar una consola serie y usarla como pantalla para poder ver el proceso de _boot_. Esto se consigue deshabilitando la salida VGA y redirigiendo la consola web al puerto serie indicado.

* Expandir el centro de datos (_Datacenter_)
* Expandir el servidor (_node_): `pve`
* Seleccionar la máquina virtual: `700 (ubuntu-2204-template)`
* Selecciónar la opción **Hardware**
* Pulsar el botón **Add**
* Selecciónar la opción `Serial Port`
  * Serial Port: `0`
* Editar el valor del **Display**
  * Graphic card: `Serial terminal 0`

## A6. Descargar la _cloud image_ de Ubuntu

La descarga de la imagen desde la web de Ubuntu se realiza utilizando el comando `wget` desde una _shell_ de Proxmox.

* Expandir el centro de datos (_Datacenter_)
* Expandir el servidor (_node_): `pve`
* Seleccionar la opción **Shell**
* Ejecutar el siguiente comando:

```
wget https://cloud-images.ubuntu.com/minimal/releases/jammy/release/ubuntu-22.04-minimal-cloudimg-amd64.img
```

## A7. Cambiar el tamaño de la _cloud image_

El tamaño de la imagen descargada desde Ubuntu es de **2.2GB** en formato `qcow2` tal como se puede comprobar utilizando el comando `qemu-img`:

```
root@pve:~# qemu-img info ubuntu-22.04-minimal-cloudimg-amd64.img 
image: ubuntu-22.04-minimal-cloudimg-amd64.img
file format: qcow2
virtual size: 2.2 GiB (2361393152 bytes)
disk size: 284 MiB
cluster_size: 65536
Format specific information:
    compat: 0.10
    compression type: zlib
    refcount bits: 16
Child node '/file':
    filename: ubuntu-22.04-minimal-cloudimg-amd64.img
    protocol type: file
    file length: 284 MiB (297664512 bytes)
    disk size: 284 MiB
```

En esta plantilla se cambiará el tamaño de la imagen a **16GB** para que todas las máquinas virtuales que se creen a partir de la misma tengan ese tamaño de disco.

* Expandir el centro de datos (_Datacenter_)
* Expandir el servidor (_node_): `pve`
* Seleccionar la opción **Shell**
* Ejecutar los siguientes comandos:

```
qemu-img resize ubuntu-22.04-minimal-cloudimg-amd64.img 16G
```

Si todo funciona correctamente, el tamaño de la imagen cambiará al tamaño indicado:

```
root@pve:~# qemu-img info ubuntu-22.04-minimal-cloudimg-amd64.img 
image: ubuntu-22.04-minimal-cloudimg-amd64.img
file format: qcow2
virtual size: 16 GiB (17179869184 bytes)
disk size: 284 MiB
cluster_size: 65536
Format specific information:
    compat: 0.10
    compression type: zlib
    refcount bits: 16
Child node '/file':
    filename: ubuntu-22.04-minimal-cloudimg-amd64.img
    protocol type: file
    file length: 284 MiB (297665024 bytes)
    disk size: 284 MiB
```

## A8. Importar la _cloud image_ a un disco

A continuación hay que importar la _cloud image_ en la máquina virtual para poderla utilizar como disco de arranque. Después de importarla, ésta queda como _Unused Disk 0_.

* Expandir el centro de datos (_Datacenter_)
* Expandir el servidor (_node_): `pve`
* Seleccionar la opción **Shell**
* Ejecutar el siguiente comando:

```
qm importdisk 700 ubuntu-22.04-minimal-cloudimg-amd64.img local-lvm
```

## A9. Configurar el disco `scsi0`

Es necesario asignar el disco a una controladora (por ejemplo `scsi0`) y definir sus características (por ejemplo `Discard` y `SSD` para [permitir comandos TRIM](https://pve.ropafamily.com/pve-docs/chapter-qm.html#qm_hard_disk){:target="_blank"}).

* Expandir el centro de datos (_Datacenter_)
* Expandir el servidor (_node_): `pve`
* Seleccionar la máquina virtual: `700 (ubuntu-2204-template)`
* Selecciónar la opción **Hardware**
* Editar el elemento **Unused Disk 0**
  * Marcar la opción `Discard`
  * Marcar la opción `Advanced`
  * Marcar la opción `SSD emulation`
* Pulsar el botón **Add**

## A10. Cambiar el orden de arranque

Por defecto, al llegar a este punto, el nuevo disco `scsi0` no está habilitado para arrancar desde él. Será necesario habilitarlo y colocarlo en la posición que se considere oportuna para este entorno.

* Expandir el centro de datos (_Datacenter_)
* Expandir el servidor (_node_): `pve`
* Seleccionar la máquina virtual: `700 (ubuntu-2204-template)`
* Seleccionar la opción **Options**
* Editar el elemento **Boot Order**
  * Habilitar el disco `scsi0`
  * Colocarlo en segunda posición (se deja el CDROM en primera por si algún día se necesita iniciar mediante una ISO)

## A11. Convertir en plantilla

Finalmente, una vez se han configurado todas las características de la máquina virtual, ésta se puede convertir en un **Template** (se marcará como _read only_ para evitar modificaciones posteriores).

* Expandir el centro de datos (_Datacenter_)
* Expandir el servidor (_node_): `pve`
* Seleccionar la máquina virtual: `700 (ubuntu-2204-template)`
* Botón derecho y elegir la opción **Convert to template**

# B. Usando la _shell_

* Expandir el centro de datos (_Datacenter_)
* Seleccionar el servidor (_node_): `pve`
* Seleccionar la opción **Shell**

## B1. Crear una máquina virtual

* Ejecutar el siguiente comando:

```
qm create 701 --name ubuntu-2204-template \
--ostype=l26 \
--scsihw virtio-scsi-single --agent enabled=1 \
--cpu x86-64-v2-AES \
--memory 1024 \
--net0 virtio,bridge=vmbr0,firewall=1 \
-ide2 none,media=cdrom
```

## B2. Añadir una unidad de **cloud-init**

* Ejecutar el siguiente comando:

```
qm set 701 --ide0 local-lvm:cloudinit
```

## B3. Configurar las opciones de **cloud-init**

* Ejecutar el siguiente comando:

```
qm set 701 --ciuser manel \
--cipassword P@ssw0rd! \
--sshkeys id_edcsa.pub \
--ipconfig0 ip=dhcp
```

## B4. Regenerar la imagen de **cloud-init**

* Expandir el centro de datos (_Datacenter_)
* Expandir el servidor (_node_): `pve`
* Seleccionar la máquina virtual: `701 (ubuntu-2204-template)`
* Selecciónar la opción **Cloud-Init**
* Pulsar el botón **Regenerate image**

{: .box-note}
**Nota**: No he encontrado la manera de regenerar la imagen desde la línea de comandos. Si no se regenera la imagen desde la interfaz gráfica, el `ide0` no se añade al **Boot Order** de la máquina virtual.

## B5. Configurar la consola de la VM

* Expandir el centro de datos (_Datacenter_)
* Expandir el servidor (_node_): `pve`
* Seleccionar la opción **Shell**
* Ejecutar el siguiente comando:

```
qm set 701 --serial0 socket \
--vga serial0
```

## B6. Descargar la _cloud image_ de Ubuntu

* Ejecutar el siguiente comando:

```
wget https://cloud-images.ubuntu.com/minimal/releases/jammy/release/ubuntu-22.04-minimal-cloudimg-amd64.img
```

## B7. Cambiar el tamaño de la _cloud image_

* Ejecutar el siguiente comando:

```
qemu-img resize ubuntu-22.04-minimal-cloudimg-amd64.img 16G
```

## B8. Importar la _cloud image_ a un disco

* Ejecutar el siguiente comando:

```
qm importdisk 701 ubuntu-22.04-minimal-cloudimg-amd64.img local-lvm
```

## B9. Configurar el disco `scsi0`

* Ejecutar el siguiente comando:

```
qm set 701 --scsi0 local-lvm:vm-701-disk-0,discard=on,iothread=1,ssd=1
```

## B10. Cambiar el orden de arranque

* Expandir el centro de datos (_Datacenter_)
* Expandir el servidor (_node_): `pve`
* Seleccionar la máquina virtual: `701 (ubuntu-2204-template)`
* Seleccionar la opción **Options**
* Editar el elemento **Boot Order**
  * Habilitar el disco `scsi0`
  * Colocarlo en segunda posición (se deja el CDROM en primera por si algún día se necesita iniciar mediante una ISO)

{: .box-note}
**Nota**: No he encontrado la manera de cambiar el _boot order_ desde la línea de comandos.

## B11. Convertir en plantilla

* Expandir el centro de datos (_Datacenter_)
* Expandir el servidor (_node_): `pve`
* Seleccionar la opción **Shell**
* Ejecutar el siguiente comando:

```
qm template 701
```

# Uso de la plantilla

## Clonación

Una vez se ha cread la plantilla, ya se pueden crear tantas máquinas virtuales como se deseen a partir de la misma usando la opción **Clone** de Proxmox VE.

### Desde la interfaz gráfica

* Expandir el centro de datos (_Datacenter_)
* Expandir el servidor (_node_): `pve`
* Seleccionar plantilla: `700 (ubuntu-2204-template)`
* Botón derecho y elegir la opción **Clone**
* Completar los datos de la nueva máquina virtual:
  * Target node: pve
  * VM ID:  100
  * Name: `ubuntu-test`
  * Mode: `Full Clone`
  * Target Storage: `data`
  * Format: QEMU image format (qcow2)
* Pulsar el botón **Clone**
* Esperar a que finalice la creación de la VM

### Desde la _shell_

* Expandir el centro de datos (_Datacenter_)
* Expandir el servidor (_node_): `pve`
* Seleccionar la opción **Shell**
* Ejecutar el siguiente comando:

```
qm clone 701 101 --name ubuntu-test --full 1 --storage data
```

## Iniciar la máquina virtual

El proceso de creación de una máquina virtual tarda unos segundos. Una vez creada, se podrá poner en marcha y abrir una consola para ver el proceso de inicio.

{: .box-note}
**Nota**: Aunque se vea un _prompt_ de login, no hay que entrar hasta que se haya realizado el aprovisionamiento del usuario y las claves SSH.

## Iniciar sesión mediante SSH

Si las claves SSH se han configurado correctamente, se podrá iniciar sesión sin necesidad de contraseña desde el usuario que generó las claves (en este ejemplo es `manel`) usando el comando:

```
ssh manel@<ip_vm>
```

{: .box-note}
**Nota**: La dirección IP asignada a la máquina virtual se puede consultar en el servidor DHCP del entorno. Si se ha configurado un usuario y una contraseña en la plantilla, tambien se podría iniciar sesión desde la consola de la máquina virtual y consultar la dirección mediante el comando `ip addr`.

## Instalación de `qemu-guest-agent`

Para que la máquina virtual pueda ser controlada desde Proxmox (p.ej. realizar un _shutdown_ controlado, que muestre la dirección IP, etc.) es necesario instalar el [agente de QEMU](https://pve.proxmox.com/wiki/Qemu-guest-agent){:target="_blank"} mediante los siguientes comandos:

```
sudo apt install qemu-guest-agent -y
sudo reboot
```

Después de reiniciar, si todo funciona correctamente, Proxmox mostrará información acerca de la máquina virtual.
