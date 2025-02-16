---
layout: page
title: HomeLab 
blog-width: true
---

Esta página contiene notas sobre diferentes aspectos de mi **HomeLab** que no están documentados explícitamente en algún artículo del [blog](/blog) con los _tags_ [HomeLab](/tags#HomeLab), [Docker](/tags#Docker), [Raspberry](/tags#Raspberry), [Proxmox](/tags#Proxmox), etc.

| [Raspberry Pi 4](#raspberry-pi-4) | [Proxmox](#proxmox) |

# Raspberry Pi 4

## Fichero `/boot/cmdline.txt`

El fichero `/boot/cmdline.txt` es un fichero con una única línea de texto con varios parámetros separados por espacios que se utiliza para configurar los parámetros de arranque del sistema operativo.

El contenido de mi fichero es el siguiente:

```
console=serial0,115200 console=tty1 root=PARTUUID=15808b80-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait ipv6.disable=1
```

Muchos de los parámetros son bastante estándar pero yo he añadido algunas opciones como deshabilitar el protocolo IPv6:

* `console=serial0,115200`: Configura la consola serial a 115200 baudios para la comunicación.
* `console=tty1`: Configura la consola principal para mostrar mensajes de arranque en el dispositivo `tty1`.
* `root=PARTUUID=15808b80-02`: Especifica la partición raíz del sistema de archivos usando el **identificador de partición UUID**.
* `rootfstype=ext4`: Define el tipo de sistema de archivos de la partición raíz como `ext4`.
* `elevator=deadline`: Establece el planificador de E/S a `deadline`, lo que puede mejorar el rendimiento en ciertas situaciones.
* `fsck.repair=yes`: Habilita la reparación automática del sistema de archivos en caso de que se detecten errores durante el arranque.
* `rootwait`: Espera a que la partición raíz esté disponible antes de continuar con el proceso de arranque.
* `ipv6.disable=1`: **Deshabilita el soporte para IPv6**.

## Fichero `/boot/config.txt`

El fichero `/boot/config.txt` se utiliza para configurar diversos aspectos del hardware y del sistema operativo durante el proceso de arranque.

El contenido de mi fichero es el siguiente:

```
# For more options and information see
# http://rpf.io/configtxt
# Some settings may impact device functionality. See link above for details

# uncomment if you get no picture on HDMI for a default "safe" mode
#hdmi_safe=1

# uncomment this if your display has a black border of unused pixels visible
# and your display can output without overscan
#disable_overscan=1

# uncomment the following to adjust overscan. Use positive numbers if console
# goes off screen, and negative if there is too much border
#overscan_left=16
#overscan_right=16
#overscan_top=16
#overscan_bottom=16

# uncomment to force a console size. By default it will be display's size minus
# overscan.
#framebuffer_width=1280
#framebuffer_height=720

# uncomment if hdmi display is not detected and composite is being output
#hdmi_force_hotplug=1

# uncomment to force a specific HDMI mode (this will force VGA)
#hdmi_group=1
#hdmi_mode=1

# uncomment to force a HDMI mode rather than DVI. This can make audio work in
# DMT (computer monitor) modes
#hdmi_drive=2

# uncomment to increase signal to HDMI, if you have interference, blanking, or
# no display
#config_hdmi_boost=4

# uncomment for composite PAL
#sdtv_mode=2

#uncomment to overclock the arm. 700 MHz is the default.
#arm_freq=800

# Uncomment some or all of these to enable the optional hardware interfaces
#dtparam=i2c_arm=on
#dtparam=i2s=on
#dtparam=spi=on

# Uncomment this to enable infrared communication.
#dtoverlay=gpio-ir,gpio_pin=17
#dtoverlay=gpio-ir-tx,gpio_pin=18

# Additional overlays and parameters are documented /boot/overlays/README

# Enable audio (loads snd_bcm2835)
dtparam=audio=on

# Enable DRM VC4 V3D drive
# dtoverlay=vc4-kms-v3d
max_framebuffers=2
arm_64bit=1

# https://diode.io/raspberry%20pi/running-forever-with-the-raspberry-pi-hardware-watchdog-20202/
# Habilitar hardware watchdog para reiniciar la Raspberry Pi si esta se cuelga
# dtparam=watchdog=on

# https://pimylifeup.com/raspberry-pi-disable-wifi/
# Deshabilitar Wi-Fi y Bluetooth
dtoverlay=disable-wifi
dtoverlay=disable-bt
```

Muchos de los parámetros son bastante estándar pero yo he añadido algunas opciones como **deshabilitar los adaptadores Wi-Fi y Bluetooth**.

## Herramienta `pi-gen`

La herramienta [**pi-gen**](https://github.com/RPi-Distro){:target="_blank"} se utiliza para crear las imágenes oficiales de Raspberry Pi OS (las de 32 bits basadas en Raspbian y las de 64 bits basadas en Debian).

## Fichero `/etc/dhcpcd.conf`

El fichero `/etc/dhcpcd.conf` configura el comportamiento del cliente DHCP en sistemas Unix como la Raspberry Pi:

* Parámetros de una **interfaz específica**
* Asignación de **dirección IP estática**
* Configuración de los **servidores DNS**
* Definición de la **puerta de enlace predeterminada**
* Añadir rutas estáticas
* Etc.

El contenido de mi fichero es el siguiente:

```
# A sample configuration for dhcpcd.
# See dhcpcd.conf(5) for details.

# Allow users of this group to interact with dhcpcd via the control socket.
#controlgroup wheel

# Inform the DHCP server of our hostname for DDNS.
hostname

# Use the hardware address of the interface for the Client ID.
clientid
# or
# Use the same DUID + IAID as set in DHCPv6 for DHCPv4 ClientID as per RFC4361.
# Some non-RFC compliant DHCP servers do not reply with this set.
# In this case, comment out duid and enable clientid above.
#duid

# Persist interface configuration when dhcpcd exits.
persistent

# Rapid commit support.
# Safe to enable by default because it requires the equivalent option set
# on the server to actually work.
option rapid_commit

# A list of options to request from the DHCP server.
option domain_name_servers, domain_name, domain_search, host_name
option classless_static_routes
# Respect the network MTU. This is applied to DHCP routes.
option interface_mtu

# Most distributions have NTP support.
#option ntp_servers

# A ServerID is required by RFC2131.
require dhcp_server_identifier

# Generate SLAAC address using the Hardware Address of the interface
#slaac hwaddr
# OR generate Stable Private IPv6 Addresses based from the DUID
slaac private

# Example static IP configuration:
interface eth0
static ip_address=192.168.1.50/24
#static ip6_address=xxxx:xxxx:xxxx:xxxx::xx/64
static routers=192.168.1.1
#static domain_name_servers=9.9.9.9 149.112.112.112
static domain_name_servers=1.1.1.1 1.0.0.1

# It is possible to fall back to a static IP if DHCP fails:
# define static profile
#profile static_eth0
#static ip_address=192.168.1.50/24
#static routers=192.168.1.1
#static domain_name_servers=1.1.1.1 1.0.0.1

# fallback to static profile on eth0
#interface eth0
#fallback static_eth0

# Evitar DHCP en las interfaces virtuales de los Docker (DHCPCD route socket overflowed)
# https://github.com/raspberrypi/linux/issues/4092
denyinterfaces veth*
```

# Proxmox
