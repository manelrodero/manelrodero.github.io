---
layout : post
blog-width: true
title: 'Plantilla de Debian 13 en Proxmox VE 8.4.6'
date: '2025-08-11 20:51:13'
#last-updated: '2025-08-11 20:51:13'
published: true
tags:
- Proxmox
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2025-08-11_cover_2.png"
thumbnail-img: ""
---

Una de las acciones más repetitivas que se realizan en [Proxmox VE](https://www.proxmox.com/en/proxmox-virtual-environment/overview){:target="_blank"} es la [creación de **máquinas virtuales**](https://pve.proxmox.com/pve-docs/chapter-qm.html#qm_virtual_machines_settings){:target="_blank"}. Cada vez que se crea una VM hay que definir sus parámetros generales, el sistema operativo que se quiere instalar, el tamaño y tipo del disco que tendrán, el número de CPU y memoria RAM que se asignará, la dirección IP, etc.

La [creación de una **plantilla**](https://pve.proxmox.com/pve-docs/chapter-qm.html#qm_templates){:target="_blank"} permite simplificar este proceso preparando una máquina virtual con unas determinadas características que después se marcará como _read only_ y se podrá [**clonar**](https://pve.proxmox.com/pve-docs/chapter-qm.html#qm_copy_and_clone){:target="_blank"} tantas veces como se desee.

Para crear esta plantilla se utilizará una [imagen oficial de **Debian Cloud**](https://cloud.debian.org/images/cloud/){:target="_blank"}, concretamente una [**Debian 13 Generic Cloud**](https://cloud.debian.org/images/cloud/trixie/latest/){:target="_blank"} (_Trixie_), que se desplegará utilizando [**cloud-init**](https://cloudinit.readthedocs.io/en/latest/){:target="_blank"} para inicializar el usuario y la configuración de red.

{: .box-note}
**Nota**: En esta ocasión se utilizará la _shell_ para crear esta plantilla aunque, tal como expliqué al crear la [plantilla de Ubuntu Minimal 22.04 LTS](creacion-de-una-plantilla-de-ubuntu-en-proxmox-ve), también se puede relizar desde la interfaz gráfica.

## 1. Crear una máquina virtual

* Expandir el centro de datos (_Datacenter_)
* Seleccionar el servidor (_node_): `pve`
* Seleccionar la opción **Shell**
* Ejecutar el siguiente comando:

```bash
qm create 703 --name debian-13-bios \
--ostype=l26 \
--scsihw virtio-scsi-single --agent enabled=1 \
--cpu x86-64-v2-AES \
--memory 1024 \
--net0 virtio,bridge=vmbr0,firewall=1 \
-ide2 none,media=cdrom
```

{: .box-note}
**Nota**: El comando anterior crea una VM usando el tipo de máquina `i440fx` y la BIOS `SeaBIOS`. Si se requiere **UEFI** o **PCIe passthrough** es mejor crearla usando `q35` y `OVMF (UEFI)`.

```bash
qm create 703 --name debian-13-uefi \
--machine q35 \
--bios ovmf \
--ostype l26 \
--scsihw virtio-scsi-single --agent enabled=1 \
--cpu x86-64-v2-AES \
--memory 1024 \
--net0 virtio,bridge=vmbr0,firewall=1 \
--ide2 none,media=cdrom
```

## 2. Configurar las opciones de **cloud-init**

* Ejecutar el siguiente comando:

```bash
qm set 703 --ciuser manel \
--cipassword P@ssw0rd! \
--sshkeys id_edcsa.pub \
--ipconfig0 ip=dhcp
```

## 3. Añadir una unidad de **cloud-init**

* Ejecutar el siguiente comando:

```bash
qm set 703 --ide0 local-lvm:cloudinit
```

## 4. Configurar la consola de la VM

* Ejecutar el siguiente comando:

```bash
qm set 703 --serial0 socket \
--vga serial0
```

## 5. Descargar la _cloud image_ de Ubuntu

* Ejecutar el siguiente comando:

```bash
wget https://cloud.debian.org/images/cloud/trixie/latest/debian-13-generic-amd64.qcow2
```

## 6. Cambiar el tamaño de la _cloud image_

* Ejecutar el siguiente comando:

```bash
qemu-img resize debian-13-generic-amd64.qcow2 8G
```

## 7. Importar la _cloud image_ a un disco

* Ejecutar el siguiente comando:

```bash
qm importdisk 703 debian-13-generic-amd64.qcow2 local-lvm
```

## 8. Configurar el disco `scsi0`

* Ejecutar el siguiente comando:

```bash
qm set 703 --scsi0 local-lvm:vm-703-disk-0,discard=on,iothread=1,ssd=1
```

## 9. Cambiar el orden de arranque

* Ejecutar el siguiente comando:

```bash
qm set 703 --boot "order=ide2;scsi0;net0"
```

## 10. Instalar `qemu-guest-agent`

* Ejecutar el siguiente comando para poner en marcha la VM:

```bash
qm start 703
```

* Acceder a la `Console` de la VM
* Iniciar sesión con el usuario `manel` y la contraseña configurada anteriormente
* Actualizar los repositorios, instalar `qemu-guest-agent` y borrar paquetes obsoletos:

```bash
sudo apt update
sudo apt install qemu-guest-agent -y
```

* Apagar la VM:

```bash
history -c
sudo shutdown -h now
```

## 11. Convertir en plantilla

```bash
qm template 703
```

## Crear una VM clonando la plantilla

* Ejecutar el siguiente comando:

```bash
qm clone 703 103 --name debian-test --full 1 --storage local-lvm
```

El proceso de clonación de una máquina virtual tarda unos pocos segundos.

## Iniciar la máquina virtual

Una vez creada, se podrá poner en marcha y abrir una consola para ver el proceso de inicio:

```bash
qm start 103
```

{: .box-note}
**Nota**: Aunque se vea un _prompt_ de login, no hay que entrar hasta que haya finalizado el aprovisionamiento del usuario y las claves SSH.

![Cloud Init][2]

## Iniciar sesión mediante SSH

Si las claves SSH se han configurado correctamente, se podrá iniciar sesión sin necesidad de contraseña desde el usuario que generó las claves (en este ejemplo es `manel`) usando el comando:

```bash
ssh manel@<ip_vm>
```

{: .box-note}
**Nota**: La dirección IP asignada a la máquina virtual se puede consultar en la sección `Summary` de la VM gracias a tener instalado el agente de QEMU.

### Historial de cambios

* **2025-08-11**: Documento inicial

[2]: /assets/img/blog/2025-08-11_image_2.png "Cloud Init"
