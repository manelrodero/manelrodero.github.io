---
layout : post
blog-width: true
title: 'Actualización de BOOTx64.EFI en Proxmox'
date: '2025-03-11 22:13:08'
#last-updated: '2025-03-11 22:13:08'
published: true
tags:
- Proxmox
author:
  display_name: Manel Rodero
#cover-img: "/assets/img/blog/2025-03-11_cover.png"
thumbnail-img: ""
---

Cuando estaba ejecutando `pveupgrade` para actualizar mi servidor Proxmox ha aparecido el siguiente mensaje indicando que se ha encontrado un _bootloader_ en `/boot/efi/EFI/BOOT/BOOTX64.efi`" pero los **paquetes GRUB** no están configurados para actualizarlo:

```plaintext
[...]
Processing triggers for man-db (2.11.2-2) ...
Processing triggers for mailcap (3.70+nmu1) ...
Processing triggers for pve-ha-manager (4.0.6) ...
Processing triggers for initramfs-tools (0.142+deb12u1) ...
update-initramfs: Generating /boot/initrd.img-6.8.12-8-pve
Running hook script 'zz-proxmox-boot'..
Re-executing '/etc/kernel/postinst.d/zz-proxmox-boot' in new private mount namespace..
No /etc/kernel/proxmox-boot-uuids found, skipping ESP sync.

Removable bootloader found at '/boot/efi/EFI/BOOT/BOOTX64.efi', but GRUB packages not set up to update it!
Run the following command:

echo 'grub-efi-amd64 grub2/force_efi_extra_removable boolean true' | debconf-set-selections -v -u

Then reinstall GRUB with 'apt install --reinstall grub-efi-amd64'

Processing triggers for libc-bin (2.36-9+deb12u9) ...

Your System is up-to-date


Seems you installed a kernel update - Please consider rebooting
this node to activate the new kernel.
```

Tal como nos indica el mensaje anterior, únicamente hay que **cambiar la configuración de `grub-efi-amd64`** y volver a reinstalarlo.

Usamos el comando `efibootmgr` para mostrar información detallada sobre las entradas de arranque UEFI configuradas en el firmware del sistema:

```bash
efibootmgr -v
```

```plaintext
BootCurrent: 0002
Timeout: 1 seconds
BootOrder: 0002
Boot0002* proxmox       HD(2,GPT,e7ee1e90-xxxx-xxxx-xxxx-xxxxxxxxxxxx,0x800,0x200000)/File(\EFI\proxmox\grubx64.efi)
```

En la salida anterior se puede ver:

* Entrada activa (BootCurrent): `0002`
* Tiempo de espera (Timeout): `1 seconds`
* Orden de arranque actual (BootOrder): `0002`
* Una única entrada de arranque (el asterisco indica que está activada):
  * Número de entrada: `0002`
  * Nombre descriptivo: `proxmox`
  * **HD(2)**: Segunda unidad de disco detectada por el firmware
  * **GPT**: Usa tabla de particiones GPT
  * e7ee1e90-xxxx-xxxx-xxxx-xxxxxxxxxxxx: UUID de la partición EFI
  * **0x800,0x200000**: Offset y tamaño de la partición
  * **File(\EFI\proxmox\grubx64.efi)**: El archivo `.efi` que se ejecuta para arrancar el sistema. En este caso, es el GRUB de Proxmox, ubicado en la partición EFI bajo esa ruta.

Usamos el comando `debconf-show` para mostrar las **opciones de configuración** almacenadas por Debconf para el paquete `grub-efi-amd64`, que es la versión de GRUB diseñada para sistemas UEFI en arquitectura AMD64 (x86_64):

```bash
debconf-show grub-efi-amd64
```

```plaintext
  grub2/update_nvram: true
  grub2/linux_cmdline_default: quiet
  grub2/force_efi_extra_removable: false
  grub2/enable_os_prober: false
  grub2/linux_cmdline:
  grub2/kfreebsd_cmdline:
  grub2/kfreebsd_cmdline_default: quiet
```

A continuación, modificamos la configuración para permitir que GRUB pueda actualizar los archivos `.efi`:

```bash
echo 'grub-efi-amd64 grub2/force_efi_extra_removable boolean true' | debconf-set-selections -v -u
```

```plaintext
info: Trying to set 'grub2/force_efi_extra_removable' [boolean] to 'true'
info: Loading answer for 'grub2/force_efi_extra_removable'
```

Al volver a ejecutar el comando `debconf-show` se observa la modificación realizada:

```bash
debconf-show grub-efi-amd64
```

```plaintext
  grub2/kfreebsd_cmdline:
  grub2/linux_cmdline:
  grub2/kfreebsd_cmdline_default: quiet
  grub2/update_nvram: true
  grub2/force_efi_extra_removable: true
  grub2/linux_cmdline_default: quiet
  grub2/enable_os_prober: false
```

Finalmente, se puede reinstalar GRUB:

```bash
apt install --reinstall grub-efi-amd64
```

```plaintext
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages were automatically installed and are no longer required:
  libnl-genl-3-200 libpcsclite1 proxmox-kernel-6.8.12-2-pve-signed proxmox-kernel-6.8.8-1-pve-signed
Use 'apt autoremove' to remove them.
0 upgraded, 0 newly installed, 1 reinstalled, 0 to remove and 0 not upgraded.
Need to get 0 B/45.7 kB of archives.
After this operation, 0 B of additional disk space will be used.
Preconfiguring packages ...
(Reading database ... 83089 files and directories currently installed.)
Preparing to unpack .../grub-efi-amd64_2.06-13+pmx5_amd64.deb ...
Unpacking grub-efi-amd64 (2.06-13+pmx5) over (2.06-13+pmx5) ...
Setting up grub-efi-amd64 (2.06-13+pmx5) ...
Installing for x86_64-efi platform.
File descriptor 3 (pipe:[149045]) leaked on vgs invocation. Parent PID 29634: grub-install.real
File descriptor 3 (pipe:[149045]) leaked on vgs invocation. Parent PID 29634: grub-install.real
Installation finished. No error reported.
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-6.8.12-8-pve
Found initrd image: /boot/initrd.img-6.8.12-8-pve
Found linux image: /boot/vmlinuz-6.8.12-6-pve
Found initrd image: /boot/initrd.img-6.8.12-6-pve
Found linux image: /boot/vmlinuz-6.8.12-2-pve
Found initrd image: /boot/initrd.img-6.8.12-2-pve
Found linux image: /boot/vmlinuz-6.8.8-1-pve
Found initrd image: /boot/initrd.img-6.8.8-1-pve
Found linux image: /boot/vmlinuz-6.5.13-6-pve
Found initrd image: /boot/initrd.img-6.5.13-6-pve
Found memtest86+ 64bit EFI image: /boot/memtest86+x64.efi
Adding boot menu entry for UEFI Firmware Settings ...
done
```

Se hace un poco de limpieza ejecutando el comando `apt autoremove`:

```bash
apt autoremove
```

```plaintext
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages will be REMOVED:
  libnl-genl-3-200 libpcsclite1 proxmox-kernel-6.8.12-2-pve-signed proxmox-kernel-6.8.8-1-pve-signed
0 upgraded, 0 newly installed, 4 to remove and 0 not upgraded.
After this operation, 1,154 MB disk space will be freed.
Do you want to continue? [Y/n] Y
(Reading database ... 83086 files and directories currently installed.)
Removing libnl-genl-3-200:amd64 (3.7.0-0.2+b1) ...
Removing libpcsclite1:amd64 (1.9.9-2) ...
Removing proxmox-kernel-6.8.12-2-pve-signed (6.8.12-2) ...
Examining /etc/kernel/postrm.d.
run-parts: executing /etc/kernel/postrm.d/initramfs-tools 6.8.12-2-pve /boot/vmlinuz-6.8.12-2-pve
update-initramfs: Deleting /boot/initrd.img-6.8.12-2-pve
run-parts: executing /etc/kernel/postrm.d/proxmox-auto-removal 6.8.12-2-pve /boot/vmlinuz-6.8.12-2-pve
run-parts: executing /etc/kernel/postrm.d/zz-proxmox-boot 6.8.12-2-pve /boot/vmlinuz-6.8.12-2-pve
Re-executing '/etc/kernel/postrm.d/zz-proxmox-boot' in new private mount namespace..
No /etc/kernel/proxmox-boot-uuids found, skipping ESP sync.
run-parts: executing /etc/kernel/postrm.d/zz-systemd-boot 6.8.12-2-pve /boot/vmlinuz-6.8.12-2-pve
run-parts: executing /etc/kernel/postrm.d/zz-update-grub 6.8.12-2-pve /boot/vmlinuz-6.8.12-2-pve
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-6.8.12-8-pve
Found initrd image: /boot/initrd.img-6.8.12-8-pve
Found linux image: /boot/vmlinuz-6.8.12-6-pve
Found initrd image: /boot/initrd.img-6.8.12-6-pve
Found linux image: /boot/vmlinuz-6.8.8-1-pve
Found initrd image: /boot/initrd.img-6.8.8-1-pve
Found linux image: /boot/vmlinuz-6.5.13-6-pve
Found initrd image: /boot/initrd.img-6.5.13-6-pve
Found memtest86+ 64bit EFI image: /boot/memtest86+x64.efi
Adding boot menu entry for UEFI Firmware Settings ...
done
Removing proxmox-kernel-6.8.8-1-pve-signed (6.8.8-1) ...
Examining /etc/kernel/postrm.d.
run-parts: executing /etc/kernel/postrm.d/initramfs-tools 6.8.8-1-pve /boot/vmlinuz-6.8.8-1-pve
update-initramfs: Deleting /boot/initrd.img-6.8.8-1-pve
run-parts: executing /etc/kernel/postrm.d/proxmox-auto-removal 6.8.8-1-pve /boot/vmlinuz-6.8.8-1-pve
run-parts: executing /etc/kernel/postrm.d/zz-proxmox-boot 6.8.8-1-pve /boot/vmlinuz-6.8.8-1-pve
Re-executing '/etc/kernel/postrm.d/zz-proxmox-boot' in new private mount namespace..
No /etc/kernel/proxmox-boot-uuids found, skipping ESP sync.
run-parts: executing /etc/kernel/postrm.d/zz-systemd-boot 6.8.8-1-pve /boot/vmlinuz-6.8.8-1-pve
run-parts: executing /etc/kernel/postrm.d/zz-update-grub 6.8.8-1-pve /boot/vmlinuz-6.8.8-1-pve
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-6.8.12-8-pve
Found initrd image: /boot/initrd.img-6.8.12-8-pve
Found linux image: /boot/vmlinuz-6.8.12-6-pve
Found initrd image: /boot/initrd.img-6.8.12-6-pve
Found linux image: /boot/vmlinuz-6.5.13-6-pve
Found initrd image: /boot/initrd.img-6.5.13-6-pve
Found memtest86+ 64bit EFI image: /boot/memtest86+x64.efi
Adding boot menu entry for UEFI Firmware Settings ...
done
Processing triggers for libc-bin (2.36-9+deb12u9) ...
```

Y listo, ya se puede reiniciar para que Proxmox utilice el nuevo **bootloader**.

### Historial de cambios

* **2025-03-11**: Documento inicial
