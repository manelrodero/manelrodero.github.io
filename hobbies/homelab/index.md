---
layout: page
title: HomeLab 
blog-width: true
---

Esta página contiene notas sobre diferentes aspectos de mi **HomeLab** que no están documentados explícitamente en algún artículo del [blog](/blog) con los _tags_ [HomeLab](/tags#HomeLab), [Docker](/tags#Docker), [Raspberry](/tags#Raspberry), [Proxmox](/tags#Proxmox), etc.

| [Raspberry Pi 4](#raspberry-pi-4) | [Proxmox](#proxmox) | [Unbound](#unbound) | [mDNS](#mdns) |

# Raspberry Pi 4

| [Fichero `/boot/cmdline.txt`](#fichero-bootcmdlinetxt) | [Fichero `/boot/config.txt`](#fichero-bootconfigtxt) | [Fichero `/etc/dhcpcd.conf`](#fichero-etcdhcpcdconf) | [Herramienta `pi-gen`](#herramienta-pi-gen) |

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

## Herramienta `pi-gen`

La herramienta [**pi-gen**](https://github.com/RPi-Distro){:target="_blank"} se utiliza para crear las imágenes oficiales de Raspberry Pi OS (las de 32 bits basadas en Raspbian y las de 64 bits basadas en Debian).

# Proxmox

| [Fichero `/etc/sysctl.conf`](#fichero-etcsysctlconf) | [Fichero `/etc/resolv.conf`](#fichero-etcresolvconf) |

## Fichero `/etc/sysctl.conf`

El fichero `/etc/sysctl.conf` se utiliza para configurar parámetros del núcleo (**kernel**) en tiempo de ejecución. Algunas de las configuraciones que se pueden ajustar en este archivo son:

* **Parámetros de red**: Se puede configurar el tamaño de los _buffers_ TCP/IP, habilitar/deshabilitar **IP forwarding**, ajustar el comportamiento del TCP/IP, etc.
* **Parámetros de seguridad**: Se puede activar o desactivar características de seguridad como ASLR (Address Space Layout Randomization)
* **Parámetros de memoria**: Se puede ajustar el comportamiento de la memoria swap, los límites de caché, etc.

Cada línea del archivo generalmente tiene el formato `nombre_del_parámetro = valor` y para aplicar los cambios sin reiniciar el sistema hay que ejecutar el comando **`sysctl -p`**.

En mi instalación de Proxmox he realizado los siguientes cambios:

```
# Al reiniciar por corte de red (Mensajes Bug: soft lockup - CPU#1 stuck for 320s!
# https://www.suse.com/support/kb/doc/?id=000018705
# https://wiki.debian.org/BootProcessSpeedup#Analyzing_the_boot_process
# https://www.kernel.org/doc/html/latest/admin-guide/lockup-watchdogs.html
kernel.watchdog_thresh = 20  # Soft lockup si núcleo CPU ocupado más de 20 segundos sin dejar que otros procesos se ejecuten (defecto = 10)
kernel.softlockup_panic = 1  # Si se produce un Soft Lockup, se producirá un Panic (detenerse y mostrar error crítico)
kernel.panic = 60            # Reiniciar el sistema después de 60 segundos de haberse producido el Panic

# https://www.reddit.com/r/docker/comments/z3xbd2/how_to_fix_sosndbuf_sorcvbuf_warning_unbound_on/
# Evitar error en LXC con Unbound (PiHole)
net.core.rmem_max = 4194304
```

## Fichero `/etc/resolv.conf`

El archivo `/etc/resolv.conf` se utiliza para configurar el _resolver_ de nombres de dominio (DNS) indicando, por ejemplo, los servidores DNS que el sistema usará para resolver nombres de dominio en direcciones IP o el sufijo de búsqueda que se agregará a un nombre de dominio corto para intentar resolverlo.

En mi caso, he configurado los siguientes valores desde `Datacenter` &rarr; `pve` &rarr; `System` &rarr; `DNS`:

```
search home
nameserver 1.1.1.1
nameserver 1.0.0.1
```

# Unbound

En la [instalación de Pi-hole en LXC](/blog/instalar-pihole-en-proxmox-lxc) se configuró **Unbound**.

Para evitar errores relacionados con el _buffer_ de recepción, se aumentó la cantidad máxima de memoria que el _kernel_ asigna para la recepción de datos en un _socket_.

```
# Fichero /etc/sysctl.conf del host Proxmox
net.core.rmem_max = 4194304
```

Además, para evitar mensajes de _warning_ relacionados con el módulo de caché que no se está usando se añaden las siguientes líneas al fichero `/etc/unbound/unbound.conf.d/pi-hole.conf`:

```
    # unbound-control status -> Dice que estamos usando los tres modulos, pero esta configuracion da un error:
    # warning: subnetcache: prefetch is set but not working for data originating from the subnet module cache
    #
    # Segun Copilot, se podria desactivar el Prefetch o desactivar la Cache
    # unbound-control stats -> Dice que no usamos la cache, por tanto la desactivamos
    # module-config: "subnetcache validator iterator"
    module-config: "validator iterator"
```

# mDNS

**mDNS** (Multicast DNS) es un protocolo que permite la resolución de nombres de _host_ en una red local sin necesidad de un servidor DNS central. Si se tiene habilitado mDNS en la red, los dispositivos pueden anunciar y resolver nombres con el sufijo `.local`.

En Linux (p.ej. en la Raspberry Pi 4) este protocolo es implementado por **Avahi**:

```
pi@pi4nas:~ $ dpkg -l | grep avahi-daemon
ii  avahi-daemon                         0.8-5+deb11u3                      arm64        Avahi mDNS/DNS-SD daemon
```

Se puede comprobar el estado del servicio usando `systemctl`:

```
pi@pi4nas:~ $ systemctl status avahi-daemon.service
● avahi-daemon.service - Avahi mDNS/DNS-SD Stack
     Loaded: loaded (/lib/systemd/system/avahi-daemon.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2025-01-19 22:32:36 CET; 1 months 6 days ago
TriggeredBy: ● avahi-daemon.socket
   Main PID: 337 (avahi-daemon)
     Status: "avahi-daemon 0.8 starting up."
      Tasks: 2 (limit: 9293)
        CPU: 2min 44.683s
     CGroup: /system.slice/avahi-daemon.service
             ├─337 avahi-daemon: running [pi4nas.local]
             └─344 avahi-daemon: chroot helper
```

En Windows, la resolución de nombres se realiza en el siguiente orden:

* **Caché de DNS** (se puede ver con `ipconfig /displaydns` y borrar con `ipconfig /flushdns`)
* **Archivo _hosts_** (`C:\Windows\System32\drivers\etc\hosts`)
* **DNS** (Domain Name System)
* **mDNS** (Multicast DNS), sobretodo si el nombre acaba en `.local`
* **LLMNR** (Link-Local Multicast Name Resolution), sobretodo en IPv6
* **NetBIOS**

Se puede deshabilitar mDNS mediante la siguiente clave de registro:

```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters]
"EnableMulticast"=dword:00000000
```

O añadiendo la siguiente GPO de `Computer Configuration`:

* `Administrative Templates`
  * `Network` &rarr; `DNS Client`
    * `Turn off smart multi-homed name resolution` (**Enabled**)

Después se aplicaría la política y se reiniciaría el servicio DNS:

```
gpupdate /force
net stop dnscache && net start dnscache
```

Se puede utilizar **WireShark** y filtrar los paquetes UDP/5353 usando `udp.port == 5353` para ver los anuncios mDNS en la red local.
