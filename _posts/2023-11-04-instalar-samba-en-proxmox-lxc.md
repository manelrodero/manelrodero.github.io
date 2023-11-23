---
layout : post
blog-width: true
title: 'Instalar Samba en Proxmox (LXC)'
date: '2023-11-04 20:11:11'
published: true
tags:
- Proxmox
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2023-11-04_cover.png"
thumbnail-img: ""  
---

Desde Febrero de 2021 he estado usando una [instalación de Samba en una Raspberry Pi 4](instalar-samba-en-raspberry-pi) para tener diferentes _shares_ relacionados con aplicaciones multimedia.

En esta ocasión voy a instalar Samba en un LXC de Proxmox con TurnKey Core. El procedimiento para crear este LXC es el mismo que usé para [ejecutar Docker en Proxmox LXC](docker-en-proxmox-lxc-con-turnkey-core).

# Características del LXC

Este **servidor de ficheros** con [Samba](https://www.samba.org/) se ejecutará sobre un LXC con las siguientes características:

* LXC no privilegiado, sin _nesting_ y sin `keyctl`
* CT ID 300
* Password + SSH key file
* Disco de **4GB**
* CPU con 1 _core_
* 1024MB de memoria
* IP fija 192.168.1.90
* Configurado con _Start at boot_

El archivo de configuración de esta máquina se guarda en el fichero `/etc/pve/lxc/300.conf`:

```
arch: amd64
cores: 1
hostname: samba
memory: 1024
nameserver: 192.168.1.1
net0: name=eth0,bridge=vmbr0,firewall=1,gw=192.168.1.1,hwaddr=XX:XX:XX:XX:XX:XX,ip=192.168.1.90/24,type=veth
onboot: 1
ostype: debian
rootfs: local-lvm:vm-300-disk-0,size=4G
searchdomain: home
swap: 512
unprivileged: 1
```

El archivo de configuración del contenedor LXC se guarda en el fichero `/var/lib/lxc/300/config`:

```
lxc.cgroup.relative = 0
lxc.cgroup.dir.monitor = lxc.monitor/300
lxc.cgroup.dir.container = lxc/300
lxc.cgroup.dir.container.inner = ns
lxc.arch = amd64
lxc.include = /usr/share/lxc/config/debian.common.conf
lxc.include = /usr/share/lxc/config/debian.userns.conf
lxc.seccomp.profile = /var/lib/lxc/300/rules.seccomp
lxc.apparmor.profile = generated
lxc.apparmor.raw = deny mount -> /proc/,
lxc.apparmor.raw = deny mount -> /sys/,
lxc.mount.auto = sys:mixed
lxc.monitor.unshare = 1
lxc.idmap = u 0 100000 65536
lxc.idmap = g 0 100000 65536
lxc.tty.max = 2
lxc.environment = TERM=linux
lxc.uts.name = samba
lxc.cgroup2.memory.max = 1073741824
lxc.cgroup2.memory.high = 1065353216
lxc.cgroup2.memory.swap.max = 536870912
lxc.rootfs.path = /var/lib/lxc/300/rootfs
lxc.net.0.type = veth
lxc.net.0.veth.pair = veth300i0
lxc.net.0.hwaddr = XX:XX:XX:XX:XX:XX
lxc.net.0.name = eth0
lxc.net.0.mtu = 1500
lxc.net.0.script.up = /usr/share/lxc/lxcnetaddbr
lxc.cgroup2.cpuset.cpus =
```

Se puede observar el **mapeo de identificadores** que se realiza entre el _host_ y el contenedor al tratarse de un [LXC no privilegiado](https://forum.proxmox.com/threads/privileged-versus-unprivileged-lxc.41733/). El ID 0 del LXC, es decir `root:root`, se mapea como el usuario `100000:100000` sin privilegios en Proxmox:

```
lxc.idmap = u 0 100000 65536
lxc.idmap = g 0 100000 65536
```

También se observa que, al no utilizar [_nesting_](https://forum.proxmox.com/threads/question-on-nested-option-lxc-container.86497/), los directorios `/proc` y `/sys` del _host_ Proxmox no se montan en el LXC:

```
lxc.apparmor.profile = generated
lxc.apparmor.raw = deny mount -> /proc/,
lxc.apparmor.raw = deny mount -> /sys/,
```

# Preparación del _host_ Proxmox

En el _host_ Proxmox hay que crear y configurar el directorio que proporcionará almacenamiento persistente de los datos y que después se montará en el LXC.

En el caso de [mi servidor Proxmox 8.0.2](proxmox-ve-802-en-un-dell-optiplex-7050), se utilizará el disco `local-lvm` de tipo [LVM](https://es.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)) ya que, desafortunadamente, no dispongo de [ZFS](https://es.wikipedia.org/wiki/ZFS_(sistema_de_archivos)).

```Bash
# Crear el directorio base para compartir con LXC (permisos 755)
mkdir /data/samba

# Cambiar el propietario del directorio según el mapeo de los LXC no privilegiados
chown -R 100000:100000 /data/samba

# Añadir el punto de montaje al contenedor LXC (el directorio en LXC se crea automáticamente)
pct set 300 -mp0 /data/samba,mp=/mnt/samba
```

{: .box-note}
**Nota**: Si no se cambia el propietario `root:root` del directorio `/data/samba` por el usuario `100000:100000`, el directorio `/mnt/samba` del LXC pertenecerá al usuario `nobody:nogroup` y habrá problemas de acceso debido a los permisos 755 del mismo.

A continuación se pone en marcha el LXC desde la CLI de Proxmox y se procede a configurarlo de la misma manera que ya expliqué en el artículo sobre [LXC y Docker](docker-en-proxmox-lxc-con-turnkey-core):

```
pct start 300
```

# Acceder al LXC

Se puede acceder al LXC utilizando la opción `Console` de la GUI o, mucho mejor, mediante **SSH** gracias a la configuración de las claves públicas que se hizo en el mismo:

```Bash
ssh root@192.168.1.90
```

# Instalación de Samba

La instalación de Samba en una distribución basada en Debian es muy sencilla usando el comando `apt`:

```Bash
apt update
apt install samba samba-common-bin -y
```

Se puede comprobar que el servicio Samba se ha puesto en marcha correctamente y que se ha configurado para hacerlo al iniciar la máquina mediante el comando `systemctl status smbd.service`:

```
* smbd.service - Samba SMB Daemon
     Loaded: loaded (/lib/systemd/system/smbd.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2023-11-05 08:31:41 CET; 35s ago
       Docs: man:smbd(8)
             man:samba(7)
             man:smb.conf(5)
    Process: 225 ExecStartPre=/usr/share/samba/update-apparmor-samba-profile (code=exited, status=0/SUCCESS)
   Main PID: 229 (smbd)
     Status: "smbd: ready to serve connections..."
      Tasks: 4 (limit: 18778)
     Memory: 30.1M
        CPU: 82ms
     CGroup: /system.slice/smbd.service
             |-229 /usr/sbin/smbd --foreground --no-process-group
             |-271 /usr/sbin/smbd --foreground --no-process-group
             |-272 /usr/sbin/smbd --foreground --no-process-group
             `-275 /usr/sbin/smbd --foreground --no-process-group

Nov 05 08:31:41 samba systemd[1]: Starting Samba SMB Daemon...
Nov 05 08:31:41 samba systemd[1]: Started Samba SMB Daemon.
```

Como en mi entorno no utilizo `NetBIOS`, desactivo el servicio [`nmbd.service`](https://www.samba.org/samba/docs/current/man-html/nmbd.8.html) para liberar recursos.

```Bash
systemctl stop nmbd.service
systemctl disable nmbd.service
systemctl status nmbd.service
```
```
* nmbd.service - Samba NMB Daemon
     Loaded: loaded (/lib/systemd/system/nmbd.service; disabled; vendor preset: enabled)
     Active: inactive (dead)
       Docs: man:nmbd(8)
             man:samba(7)
             man:smb.conf(5)
```

# Configuración de Samba

A continuación se configura Samba de forma similar a como lo hice anteriormente en [mi instalación de Samba en una Raspberry Pi](instalar-samba-en-raspberry-pi).

## Usuarios y directorios

```Bash
# Creación de directorios que Samba exportará por SMB
cd /mnt/samba
mkdir homes
mkdir media
mkdir downloads

# Cambiar el propietario de los directorios al grupo de Samba
# sambashare: LXC (111) --> Host (100111)
chown -R :sambashare homes
chown -R :sambashare media
chown -R :sambashare downloads

# Cambiar los permisos de grupo de los directorios comunes
chmod -R g+w media
chmod -R g+w downloads

# Crear mi usuario de Linux (sin login) y su directorio protegido en homes
# manel: LXC (1000:111) --> Host (101000:100111)
mkdir homes/manel
adduser --home /mnt/samba/homes/manel --no-create-home --shell /usr/bin/nologin --ingroup sambashare manel
chown manel:sambashare homes/manel
chmod 2770 homes/manel

# Crear mi usuario de Samba y habilitarlo
smbpasswd -a manel
smbpasswd -e manel

# Añadir usuarios al grupo de Samba
usermod -aG sambashare manel
usermod -aG sambashare $(whoami)
```

## Modificación de `smb.conf`

Antes de modificar el fichero de configuración de Samba en `/etc/samba/smb.conf` se hace una copia de seguridad:

```Bash
cd /etc/samba
mv smb.conf smb.conf.original
```

A continuación se edita el fichero `smb.conf` con el siguiente contenido:

```
[global]
   server string = samba.home
   server role = standalone server
   interfaces = eth0
   bind interfaces only = yes
   disable netbios = yes
   smb ports = 445
   log file = /var/log/samba/smb.log
   max log size = 10000
   security = user

[manel]
   path = /mnt/samba/homes/manel
   browseable = yes
   read only = no
   force create mode = 0660
   force directory mode = 2770
   valid users = manel

[media]
   path = /mnt/samba/media
   browseable = yes
   read only = no
   force create mode = 0660
   force directory mode = 2770
   valid users = @sambashare

[downloasd]
   path = /mnt/samba/downloads
   browseable = yes
   read only = no
   force create mode = 0660
   force directory mode = 2770
   valid users = @sambashare
```

Finalmente se reinicia el servicio `smbd.service`:

```
systemctl restart smbd.service
```

Si todo funciona correctamente, se debería poder acceder a `\\192.168.1.90` desde otra máquina. En la siguiente captura de pantalla se muestran los _shares_ vistos desde una máquina Windows:

![Samba Shares][1]

# Referencias

* [Nesting option in LXC container](https://forum.proxmox.com/threads/question-on-nested-option-lxc-container.86497/)
* [Privileged versus Unprivileged LXC](https://forum.proxmox.com/threads/privileged-versus-unprivileged-lxc.41733/)

[1]: /assets/img/blog/2023-11-04_image_1.png "Samba Shares"
