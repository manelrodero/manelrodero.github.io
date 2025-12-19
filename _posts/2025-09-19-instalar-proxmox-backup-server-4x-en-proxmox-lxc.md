---
layout : post
blog-width: true
title: 'Instalar Proxmox Backup Server 4.x en Proxmox LXC'
date: '2025-09-19 20:22:24'
last-updated: '2025-12-19 20:05:24'
published: true
tags:
- Proxmox
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2025-09-19_cover.png"
thumbnail-img: ""
---

[**Proxmox Backup Server** (PBS)](https://www.proxmox.com/en/products/proxmox-backup-server/overview){:target="_blank"} es una solución de copias de seguridad diseñada para entornos virtualizados, especialmente Proxmox VE. Permite backups incrementales, verificación de integridad, compresión y deduplicación, todo gestionado desde una interfaz web.

Instalarlo en un **contenedor LXC** dentro de Proxmox es una opción ligera y eficiente. Reduce el consumo de recursos frente a una VM, acelera el arranque y permite aprovechar discos `ext4` o `btrfs` sin complicaciones.

## Crear LXC `pbs-lxc`

La creación del contenedor `pbs-lxc` con identificador `400` utiliza parte de los _scripts_ usados para crear una plantilla de [LXC+Docker usando Debian 13](plantilla-de-lxc-con-debian-13-para-ejecutar-docker-en-proxmox).

Se creará un contenedor LXC con las siguientes características _hardware_:

* CPU: 2
* RAM: 2GB
* Disco: 10GB

```bash
clear

RED='\033[1;31m'
GREEN='\033[1;32m'
CYAN='\033[1;36m'
NC='\033[0m'

lxcname=pbs-lxc
ct_id=400

while true; do
  echo -e "${GREEN}Introduce el password de 'root':${NC}"
  read -s -p "> " password1
  echo
  echo -e "${GREEN}Confirma el password de 'root':${NC}"
  read -s -p "> " password2
  echo
  if [ "$password1" == "$password2" ]; then
    password=$password1
    break
  else
    echo -e "${RED}Error: Los passwords no coinciden.${NC}"
  fi
done

valid_ip() {
  [[ "$1" =~ ^([0-9]{1,3}\.){3}[0-9]{1,3}$ ]] || return 1
  IFS='.' read -r o1 o2 o3 o4 <<< "$1"
  for octet in "$o1" "$o2" "$o3" "$o4"; do
    (( octet >= 0 && octet <= 255 )) || return 1
  done
  return 0
}

while true; do
  echo -e "${GREEN}Introduce la dirección IP:${NC}"
  read -p "> " ip
  if valid_ip "$ip"; then
    ip="$ip/24"
    break
  else
    echo -e "${RED}Formato de IP inválido.${NC}"
  fi
done

while true; do
  echo -e "${GREEN}Introduce la puerta de enlace:${NC}"
  read -p "> " gw
  if valid_ip "$gw"; then
    break
  else
    echo -e "${RED}Formato de IP inválido.${NC}"
  fi
done

echo -e "${CYAN}Creando el contenedor ${ct_id}...${NC}"
pct create "$ct_id" local:vztmpl/debian-13-standard_13.1-1_amd64.tar.zst \
  --ostype debian --arch amd64 \
  --hostname "$lxcname" --unprivileged 1 \
  --password "$password" --ssh-public-keys /root/id_ed25519.pub \
  --storage local-lvm --rootfs local-lvm:10 \
  --cores 2 \
  --memory 2048 --swap 512 \
  --net0 name=eth0,bridge=vmbr0,firewall=1,ip="$ip",gw="$gw" \
  --features nesting=1 \
  --onboot 1 \
  --start 1
```

## Configurar LXC

```bash
pct enter $ct_id
```

### Locale

```bash
sed -i 's/^# *\(en_US.UTF-8 UTF-8\)/\1/' /etc/locale.gen
sed -i 's/^# *\(es_ES.UTF-8 UTF-8\)/\1/' /etc/locale.gen
locale-gen
cat <<EOF > /etc/default/locale
LANG=en_US.UTF-8
LANGUAGE=en_US:en
LC_CTYPE=es_ES.UTF-8
EOF
```

### Timezone

```bash
ln -sf /usr/share/zoneinfo/Europe/Madrid /etc/localtime
echo "Europe/Madrid" > /etc/timezone
dpkg-reconfigure -f noninteractive tzdata
```

### `.bashrc`

```bash
# Forzar colores en PROMPT y grep
sed -i 's/^#force_color_prompt=yes/force_color_prompt=yes/' /etc/skel/.bashrc
sed -i -E "s/^([[:space:]]*)#([[:space:]]*)alias grep='grep --color=auto'/\1\2alias grep='grep --color=auto'/" /etc/skel/.bashrc

# Aplicar también al usuario root
cp ~/.bashrc ~/.bashrc.bak
cp ~/.profile ~/.profile.bak
cp /etc/skel/.bashrc ~/.bashrc
cp /etc/skel/.profile ~/.profile

# Logout per inactivitat
cat <<EOF >> /etc/bash.bashrc

# Logout per inactivitat 30 minuts
export TMOUT=1800
readonly TMOUT
EOF
```

## Instalar PBS

Para instalar Proxmox Backup Server en Debian 13 hay que añadir el repositorio correcto:

```bash
# 1. Descargar y guardar la clave GPG en el keyring estándar
wget -q https://enterprise.proxmox.com/debian/proxmox-release-trixie.gpg -O /usr/share/keyrings/proxmox-archive-keyring.gpg

# 2. Crear el fichero de repositorio en formato .sources
cat <<EOF > /etc/apt/sources.list.d/proxmox.sources
Types: deb
URIs: http://download.proxmox.com/debian/pbs
Suites: trixie
Components: pbs-no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
EOF

# 3. Actualizar los índices de paquetes
apt update

# 4. Instalar PBS
apt install proxmox-backup-server -y

# 5. Deshabilitar repositorio pbs-enterprise
sed -i 's/^/#/' /etc/apt/sources.list.d/pbs-enterprise.sources

# 6. Deshabilitar nag screen
echo "DPkg::Post-Invoke { \"if [ -s /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js ] && ! grep -q -F 'NoMoreNagging' /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js; then sed -i '/data\\.status/{s/\\!//;s/active/NoMoreNagging/}' /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js; fi\" };" >/etc/apt/apt.conf.d/no-nag-script
```

## Actualizar LXC

```bash
apt update
apt dist-upgrade -y
```

## (Opcional) Instalar herramientas

```bash
apt install htop -y
apt install net-tools -y
```

## Añadir almacenamiento

Como mi servidor [Dell OptiPlex 7050 Micro](mini-pc-dell-optiplex-7050-micro) ya tiene ocupado todo el espacio interior para almacenamiento, utilizaré uno de los puertos USB disponibles para conectar una **unidad SSD SATA** [Crucial MX500 1TB](https://amzn.to/4oq2Z33){:target="_blank"} que tenía por casa:

* 4 puertos de tipo A en la parte posterior:
  * [Teclado inalámbrico Omoton](https://amzn.to/3XS4Hzg){:target="_blank"} (`Areson Technology Corp 2.4G Receiver`)
  * [Coordinador Zigbee Sonoff](https://amzn.to/4rmsJjD){:target="_blank"} (`Silicon Labs CP210x UART Bridge`)
  * [Adaptador Bluetooth Hideez](https://amzn.to/4iu5W16){:target="_blank"} (`Cambridge Silicon Radio, Ltd Bluetooth Dongle`)
  * [Acelerador USB Coral](https://amzn.to/48DtYmR){:target="_blank"} (`Global Unichip Corp.`)
* 1 puerto de tipo A en la parte frontal &larr; **Crucial MX500**
* 1 puerto de tipo C en la parte frontal

He metido el Crucial MX500, una unidad SSD SATA con tecnología 3D NAND y hasta 560 MB/s, dentro de una **carcasa** [Inateck FE2014-BK](https://amzn.to/4851RwS){:target="_blank"} USB 3.1 Gen 2, que soporta UASP y permite hasta 6Gbps para obtener el máximo rendimiento.

El comando `lsusb` permite ver que este dispositivo (`VIA Labs, Inc. VL817 SATA Adaptor`) está funcionando en **modo óptimo** (`uas`, `5000M`):

```
root@pve:~# lsusb
Bus 002 Device 002: ID 1a6e:089a Global Unichip Corp.
Bus 002 Device 003: ID 2109:0715 VIA Labs, Inc. VL817 SATA Adaptor
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 004: ID 0a12:0001 Cambridge Silicon Radio, Ltd Bluetooth Dongle (HCI mode)
Bus 001 Device 003: ID 10c4:ea60 Silicon Labs CP210x UART Bridge
Bus 001 Device 002: ID 25a7:fa23 Areson Technology Corp 2.4G Receiver
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

root@pve:~# lsusb -t
/:  Bus 02.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/10p, 5000M
    |__ Port 1: Dev 3, If 0, Class=Mass Storage, Driver=uas, 5000M
    |__ Port 3: Dev 2, If 0, Class=Application Specific Interface, Driver=, 5000M
/:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/16p, 480M
    |__ Port 3: Dev 2, If 0, Class=Human Interface Device, Driver=usbhid, 12M
    |__ Port 3: Dev 2, If 1, Class=Human Interface Device, Driver=usbhid, 12M
    |__ Port 5: Dev 3, If 0, Class=Vendor Specific Class, Driver=cp210x, 12M
    |__ Port 7: Dev 4, If 0, Class=Wireless, Driver=usbfs, 12M
    |__ Port 7: Dev 4, If 1, Class=Wireless, Driver=usbfs, 12M
```

El comando `dmesg` muestra la siguiente información al conectar la carcasa/disco:

```
[ 7448.292023] usb 2-1: new SuperSpeed USB device number 3 using xhci_hcd
[ 7448.312203] usb 2-1: New USB device found, idVendor=2109, idProduct=0715, bcdDevice= 0.00
[ 7448.312208] usb 2-1: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[ 7448.312210] usb 2-1: Product: HDD Enclosure
[ 7448.312211] usb 2-1: Manufacturer: Inateck Technology Inc
[ 7448.312213] usb 2-1: SerialNumber: XXXXXXXXXXXX
[ 7448.319919] usbcore: registered new interface driver usb-storage
[ 7448.325862] scsi host4: uas
[ 7448.325959] usbcore: registered new interface driver uas
[ 7448.441980] scsi 4:0:0:0: Direct-Access     CT1000MX 500SSD1          M3CR PQ: 0 ANSI: 6
[ 7448.446357] sd 4:0:0:0: Attached scsi generic sg1 type 0
[ 7448.447850] sd 4:0:0:0: [sdb] 1953525168 512-byte logical blocks: (1.00 TB/932 GiB)
[ 7448.448005] sd 4:0:0:0: [sdb] Write Protect is off
[ 7448.448008] sd 4:0:0:0: [sdb] Mode Sense: 2f 00 00 00
[ 7448.448264] sd 4:0:0:0: [sdb] Write cache: enabled, read cache: enabled, doesn't support DPO or FUA
[ 7448.473248] sd 4:0:0:0: [sdb] Preferred minimum I/O size 4096 bytes
[ 7448.473253] sd 4:0:0:0: [sdb] Optimal transfer size 33553920 bytes not a multiple of preferred minimum block size (4096 bytes)
[ 7449.897679]  sdb: sdb1
[ 7449.897881] sd 4:0:0:0: [sdb] Attached SCSI disk
```

La información anterior indica que:

* El disco ha negociado correctamente a **SuperSpeed (USB 3.x, 5 Gbps)** con el controlador xHCI
* Está usando **UASP (USB Attached SCSI Protocol)**, que es lo ideal para SSDs: menor latencia y mejor rendimiento que el modo BOT (Bulk‑Only Transport)
* El sistema detecta correctamente la unidad Crucial MX500 de 1 TB
* El kernel ha activado caché de escritura y lectura, lo que mejora el rendimiento
* El disco trabaja con bloques de 4 KiB, estándar en SSDs modernos
* El disco se ha registrado como `/dev/sdb` y está listo para usarse

Este disco se particionó dejando un 10% de espacio libre para OP tal como muestra el comando `fdisk`:

```
oot@pve:~# fdisk -l /dev/sdb
Disk /dev/sdb: 931.51 GiB, 1000204886016 bytes, 1953525168 sectors
Disk model: 500SSD1
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX

Device     Start        End    Sectors   Size Type
/dev/sdb1   2048 1758463999 1758461952 838.5G Linux LVM
```

Con esta configuración, se ha ejecutado una simple prueba de escritura y lectura utilizando el comando `dd` con diferentes sistemas de ficheros y se ha visto que el rendimiento es prácticamente el mismo:

| Sistema de ficheros | Características | Escritura secuencial | Lectura secuencial |
| --- | --- | --- | --- |
| ext4 | Simplicidad y rendimiento estable | 375 MB/s | 425 MB/s |
| btrfs | Snapshots, integridad y flexibilidad en backups | 368 MB/s | 389 MB/s |

### Sistema de ficheros **`ext4`**

Se ha decidido usar el sistema de ficheros **`ext4`** por su simplicidad. Ahora que la carcasa Inateck está conectada directamente a un puerto USB sin pasar por el [hub USB-C Orico](https://amzn.to/4p85los){:target="_blank"} no debería haber _timeouts_ que afecten a la integridad de los datos:

```
mkfs.ext4 -F -L backups /dev/sdb1
```
```
mke2fs 1.47.0 (5-Feb-2023)
/dev/sdb1 contains a btrfs file system labelled 'backups'
Creating filesystem with 219807744 4k blocks and 54951936 inodes
Filesystem UUID: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968,
        102400000, 214990848

Allocating group tables: done
Writing inode tables: done
Creating journal (262144 blocks): done
Writing superblocks and filesystem accounting information: done
```

### Montaje en `/backups`

El **UUID** mostrado por el proceso de formateo (o el obtenido mediante el comando `blkid /dev/sdb1`) se utiliza para definir el montaje de la partición en el fichero `/etc/fstab` añadiendo la siguiente línea:

```
UUID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" /backups btrfs defaults,noatime,discard,ssd,nofail 0 0
```

Después se ejecuta el comando `mount -a` y la partición `/dev/sdb1` queda montada en el directorio `/backups`.

## Configurar PBS

### Acceder al servidor

Si PBS se ha instalado correctamente, se podrá utilizar la URL `https://<ip>:8007` para acceder a la consola y proceder a configurarlo.

### Definir _storage_

Para poder usar la unidad SSD externa como _storage_ de PBS, se creará el directorio `/backups/pbs` y se montará en el contenedor LXC usando los siguientes comandos:

```bash
mkdir /backups/pbs
chown 100034:100034 /backups/pbs
pct set 400 -mp0 /backups/pbs,mp=/mnt/backups
pct reboot 400
```

Desde la consola de PBS se crea un **datastore** en el directorio que se ha montado desde Proxmox:

* Datastore &rarr; Add Datastore
  * Name: `backups`
  * Datastore type: Local
  * Backing path: `/mnt/backups`
  * GC Schedule: `daily`
  * Prune Schedule: `daily`

### Crear usuario

A continuación se crea un usuario `backups` que tendrá permisos sobre ese repositorio:

* Configuration &rarr; Access Control &rarr; User Management: Add
  * User name: `backups`
  * Password: `********`
* Configuration &rarr; Access Control &rarr; Permissions: Add &rarr; User Permission
  * Path: `/datastore/backups`
  * User: `backups@pbs`
  * Role: **DatastoreAdmin**

### Configurar _jobs_

Uno de los pasos más importantes a la hora de utilizar PBS es definir los trabajos de **Prune**, **GC** y **Verify**.

* Datastore &rarr; backups &rarr; Prune & GC Jobs: Edit
  * Garbage Collect Jobs (`daily`)
    * Schedule: `04:00`
  * Prune Jobs (`default-backups-xxxxxxxx-xxxx-xx`)
    * Schedule: `03:30`
    * Keep Daily: 14
    * Keep Weekly: 8
    * Keep Monthly: 6
* Datastore &rarr; backups &rarr; Verify Jobs: Add
  * Verify Job (`v-xxxxxxxx-xxxx`)
    * Schedule: `sun 05:00`

## Conectar Proxmox a PBS

* Datacenter &rarr; Storage &rarr; Add &rarr; Proxmox Backup Server
  * ID: `pbs-lxc`
  * Server: `<ip>`
  * Username: `backups@pbs`
  * Password: `********`
  * Datastore: `backups`
  * Fingerprint: `<key>`

**Nota**: El _fingerprint_ se puede obtener desde el **Dashboard** de PBS.

## Configurar Backups

* Datacenter &rarr; Backup &rarr; Add
  * Storage: `pbs-lxc`
  * Schedule: `03:00`
  * Selection mode: `Include selected VMs`
  * Mode: `Snapshot`

A continuación, se selecciona el _job_ y se ejecuta pulsando el botón **Run now** para probarlo.

## Añadir a Homepage

La configuración del [_widget_ de PBS](https://gethomepage.dev/widgets/services/proxmoxbackupserver/){:target="_blank"} en mi [_dashboard_ Homepage](https://gethomepage.dev/){:target="_blank"} es bastante sencilla:

* Configuration &rarr; Access Control &rarr; User Management: Add
  * User name: `api`
  * Realm: Proxmox Backup authentication server
  * Password: `********` 
* Configuration &rarr; Access Control &rarr; API Token: Add
  * User: `api@pbs`
  * Token Name: `homepage`

Este paso genera un **Token Secret** que hay que copiar y guardar:

* Token ID: `api@pbs!homepage`
* Secret: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`

* Configuration &rarr; Permissions &rarr; API Token: Add User Permission
  * Path: `\`
  * User: `api@pbs`
  * Role: Audit
  * Propagate: Yes
* Configuration &rarr; Permissions &rarr; API Token: Add API Token Permission
  * Path: `\`
  * API Token: `api@pbs!homepage`
  * Role: Audit
  * Propagate: Yes

## Referencias

* [(Casi) Todo sobre Proxmox Backup Server](https://www.youtube.com/watch?v=sws3iNGKsXs){:target="_blank"}, Jonatan Castro

### Historial de cambios

* **2025-09-19**: Documento inicial
* **2025-11-29**: Revisión y detalles sobre el almacenamiento
* **2025-11-30**: Cambio ECDSA a ED25519
* **2025-12-19**: PBS en Homepage
