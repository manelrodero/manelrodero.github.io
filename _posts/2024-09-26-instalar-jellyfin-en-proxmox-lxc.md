---
layout : post
blog-width: true
title: 'Instalar Jellyfin en Proxmox LXC'
date: '2024-09-26 18:18:47'
last-updated: '2024-09-28 12:18:47'
published: true
tags:
- Proxmox
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2024-09-26_cover.png"
thumbnail-img: ""
---

[Jellyfin](https://jellyfin.org/){:target="_blank"} es una plataforma multimedia que permite gestionar y transmitir contenido desde un servidor personal a cualquier dispositivo. Es una alternativa de código abierto a soluciones propietarias como [Emby](https://emby.media/){:target="_blank"} y [Plex](https://www.plex.tv/){:target="_blank"}.

Se puede instalar de forma sencilla en Proxmox usando contenedores ligeros LXC creados mediante los [Proxmox VE Helper-Scripts](https://tteck.github.io/Proxmox/){:target="_blank"}.

# Creación del LXC

La creación del LXC de Jellyfin se realiza desde la _shell_ del servidor Proxmox:

```
mkdir -p ~/tteck && cd ~/tteck
wget -nv https://github.com/tteck/Proxmox/raw/main/ct/jellyfin.sh
# Revisar el script ;-)
bash jellyfin.sh
```

Al ejecutar el _script_ anterior, aparecerá el asistente de instalación:

* This will create a New Jellyfin LXC. Proceed? `Yes`
* Use Default Settings? `Advanced`
* To make a selection, use the Spacebar. `Ok`
* If the default Linux distribution is not adhered to, script support will be discontinued (ubuntu 22.04). `Ok`
* Choose Distribution: **ubuntu**
* Choose version: **22.04 Jammy**
* Choose Type: **1 Unprivileged**
* Set Root Password (needed for root ssh access): **xxxxxxxx**
* Verify Root Password: **xxxxxxxx**
* Set Container ID: **310**
* Set Hostname: **jellyfin**
* Set Disk Size in GB: **8**
* Allocate CPU Cores: **2**
* Allocate RAM in MiB: **2048**
* Set a Bridge: **vmbr0**
* Set a Static IPv4 CIDR Address (/24): **192.168.1.87/24**
* Set gateway IP address: **192.168.1.1**
* Set APT-Cacher IP (leave blank for default):
* Disable IPv6: `Yes`
* Set Interface MTU Size (leave blank for default):
* Set a DNS Search Doamin (leave blank for HOST): **home**
* Set a DNS Server IP (leave blank for HOST): **192.168.1.81,192.168.1.82**
* Set a MAC Address (leave blank for default):
* Set a Vlan (leave blank for default):
* Enable Root SSH Access? `No`
* Enable Verbose Mode? `Yes`
* Ready to create Jellyfin LXC? `Yes`

{: .box-note}
Deshabilitar IPv6 implica añadir la siguiente configuración al LXC: `net.ipv6.conf.all.disable_ipv6 = 1`

Antes de poner en marcha el LXC, se pueden deshabilitar un par de opciones que no son necesarias para el funcionamiento de Jellyfin y que han sido añadidas por el _script_ ejecutado anteriormente:

* Expandir el centro de datos (_Datacenter_)
* Expandir el servidor (_node_): `pve`
* Seleccionar el contenedor (_container_): `310 (jellyfin)`
* Seleccionar la opción **Options**
* Editar los siguientes valores:
  * **Features**:
    * Desmarcar la opción **keyctl**
    * Desmarcar la opción **Nesting**

A continuación, se reinicia el LXC mediante el comando `pct reboot 310` desde la _Shell_ de Proxmox o usando el botón `Reboot` desde la configuración del propio LXC.

# Configuración inicial

Al no haber habilitado el acceso para el usuario `root` mediante SSH, se puede acceder al LXC utilizando el botón `Console` desde la interfaz gráfica del propio LXC.

De todos modos, la configuración inicial de Jellyfin se realiza mediante su interfaz web disponible en el puerto **8096** de la IP estática configurada anteriormente ([http://192.168.1.87:8096](http://192.168.1.87:8096){:target="_blank"}):

* Welcome to Jellyfin!
  * Preferred display language: `English`
* Tell us about yourself
  * Username: `admin`
  * Password: `********`
* Set up your media libraries
  * _Saltar este paso, de momento_
* Preferred Metadata Language
  * Language: Spanish; Castilian
  * Country/Region: Spain
* Set up Remote Access
  * Allow remote connections to this server: `No`
  * Enable automatic port mapping: `No`
* You're Done!

# Añadir librería local

Después de la configuración inicial, se añade una librería local al LXC para probar que Jellyfin puede descargar los metadatos y reproducir el contenido desde la propia interfaz web o desde otros dispositivos.

Al no haber habilitado el acceso para el usuario `root` mediante SSH, se puede acceder al LXC utilizando el botón `Console` desde la configuración del propio LXC.

Se crea un directorio para esta librería local y se procede a descargar el clásico [Big Buck Bunny](http://bbb3d.renderfarming.net/download.html){:target="_blank"} en formato 4K, 3840x2160, 60fps:

```
mkdir -p "/mnt/local/Big Buck Bunny (2008)" && cd "/mnt/local/Big Buck Bunny (2008)"
curl -O "http://distribution.bbb3d.renderfarming.net/video/mp4/bbb_sunflower_2160p_60fps_normal.mp4"
```

Desde la página principal de la interfaz web (`Administration` &rarr; `Dashboard` &rarr; `Libraries`) se crea una nueva librería con la siguiente configuración:

* Add Media Library
  * Content type: `Movies`
  * Display name: `Local`
  * Folders: `/mnt/local`
  * `[X]` Enable the library
  * Preferred download language: `Spanish; Castilian`
  * Country/Region: `Spain`
  * Metadata savers: `Nfo`
  * Save artwork into media folders

{: .box-note}
Los parámetros no especificados en la configuración anterior se dejan con los valores por defecto usados por Jellyfin.

# Metadatos

Si todo funciona correctamente, Jellyfin obtendrá los **metadatos** de la película "Big Buck Bunny" desde los servidores configurados en la librería y veremos algo así:

![Metadatos][1]

# Reproducción

Se puede utilizar el botón &#9654; en la parte superior derecha para reproducir la película. En el _screenshot_ se muestra la información de reproducción usando la opción `Settings` &rarr; `Playback info` del reproductor de vídeo HTML de Jellyfin:

![Reproducción][2]

# Aceleración _Hardware_

Jellyfin puede realizar [**_transcoding_**](https://jellyfin.org/docs/general/administration/hardware-acceleration/){:target="_blank"} del vídeo al vuelo usando las capacidades de una tarjeta gráfica integrada o discreta instalada en el servidor. Yo he usado la [**Intel HD Graphics 630**](https://www.techpowerup.com/gpu-specs/hd-graphics-630.c2962){:target="_blank"} **integrada** en el procesador [Intel i5-7500T](https://ark.intel.com/content/www/us/en/ark/products/97121/intel-core-i5-7500t-processor-6m-cache-up-to-3-30-ghz.html){:target="_blank"} de mi servidor [Dell OptiPlex 7050 Micro](proxmox-ve-802-en-un-dell-optiplex-7050) con Proxmox.

Para ello, únicamente he tenido que [**habilitar la Intel HD Graphics 630 en Proxmox**](habilitar-intel-hd-graphics-630-en-proxmox) y hacer el _passthrough_ de la misma al contenedor LXC de Jellyfin.

Dado que este contenedor LXC ya está preconfigurado (ver más adelante) con todo lo necesario para usar esta aceleración _hardware_, únicamente hay que configurar el servidor Jellyfin para que la use:

* `Administration` &rarr; `Dashboard` &rarr; `Playback` &rarr; `Transcoding`
  * Hardware acceleration: `Intel QuickSync (QSV)`

Si se reproduce la misma película y se fuerza la resolución a `360p - 420 kbps` para simular un dispositivo no compatible con el formato original de la película en la librería, se observa como Jellyfin realiza el **transcoding** al vuelo:

![Transcoding][3]

{: .box-note}
El uso de la tarjeta gráfica se puede observar ejecutando la utilidad `intel_gpu_top` en el servidor de Proxmox.

![Intel GPU Top][4]

# TTeck Script

El contenedor LXC generado por [Proxmox VE Helper-Scripts](https://tteck.github.io/Proxmox/){:target="_blank"} realiza una serie de acciones genéricas (para todos los LXC) y específicas (para Jellyfin) que lo dejan correctamente configurado para usar la aceleración _hardware_.

* Actualiza el SO (genérico para todos los LXC)
```
apt-get update
apt-get -o Dpkg::Options::="--force-confold" -y dist-upgrade
```
* Instala dependencias
```
apt-get install -y curl
apt-get install -y sudo
apt-get install -y gpg
apt-get install -y mc
```
* Instala soporte para aceleración gráfica
```
apt-get -y install {va-driver-all,ocl-icd-libopencl1,intel-opencl-icd,vainfo,intel-gpu-tools}
```
* Si CT_TYPE=0 (Unprivileged) configura los permisos de la tarjeta gráfica
```
chgrp video /dev/dri
chmod 755 /dev/dri
chmod 660 /dev/dri/*
adduser $(id -u -n) video
adduser $(id -u -n) render
```
* Instala Jellyfin
```
VERSION="$( awk -F'=' '/^VERSION_CODENAME=/{ print $NF }' /etc/os-release )"
mkdir -p /etc/apt/keyrings
curl -fsSL https://repo.jellyfin.org/jellyfin_team.gpg.key | gpg --dearmor --yes --output /etc/apt/keyrings/jellyfin.gpg
cat <<EOF >/etc/apt/sources.list.d/jellyfin.sources
Types: deb
URIs: https://repo.jellyfin.org/${PCT_OSTYPE}
Suites: ${VERSION}
Components: main
Architectures: amd64
Signed-By: /etc/apt/keyrings/jellyfin.gpg
EOF
apt-get update
apt-get install -y jellyfin
chown -R jellyfin:adm /etc/jellyfin
sleep 10
systemctl restart jellyfin
```
* Si CT_TYPE=0 (Unprivileged) añade el usuario `jellyfin` a los grupos `video` (104) y `render` (108)
```
sed -i -e 's/^ssl-cert:x:104:$/render:x:104:root,jellyfin/' -e 's/^render:x:108:root,jellyfin$/ssl-cert:x:108:/' /etc/group
```
* Limpieza final
```
apt-get -y autoremove
apt-get -y autoclean
```

# Referencias

* [TTeck Script Create Jellyfin LXC](https://github.com/tteck/Proxmox/blob/main/ct/jellyfin.sh){:target="_blank"} 
* [TTeck Generic Build Script](https://github.com/tteck/Proxmox/blob/main/misc/build.func){:target="_blank"} 
* [TTeck Generic Install Script](https://github.com/tteck/Proxmox/blob/main/misc/install.func){:target="_blank"} 
* [TTeck Script Install Jellyfin](https://github.com/tteck/Proxmox/blob/main/install/jellyfin-install.sh){:target="_blank"} 
* [Libraries](https://jellyfin.org/docs/general/server/libraries){:target="_blank"} (Jellyfin Server Guide)
* [Hardware acceleration](https://jellyfin.org/docs/general/administration/hardware-acceleration/intel#lxc-on-proxmox){:target="_blank"} (Intel + LXC Proxmox)

### Historial de cambios

* **2024-09-26**: Documento inicial
* **2024-09-28**: Aceleración hardware y explicación de TTTeck Scripts

[1]: /assets/img/blog/2024-09-26_image_1.png "Metadatos"
[2]: /assets/img/blog/2024-09-26_image_2.png "Reproducción"
[3]: /assets/img/blog/2024-09-26_image_3.png "Transcoding"
[4]: /assets/img/blog/2024-09-26_image_4.png "Intel GPU Top"
