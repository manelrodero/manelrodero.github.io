---
layout : post
blog-width: true
title: 'Instalar Tailscale en Proxmox LXC'
date: '2025-02-24 20:57:46'
#last-updated: '2025-02-24 20:57:46'
published: true
tags:
- Proxmox
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2025-02-24_cover.png"
thumbnail-img: ""
---

[Tailscale](https://tailscale.com/){:target=_blank} es un servicio de red privada virtual (VPN) que facilita la creación de redes seguras y privadas entre dispositivos, usuarios y servicios.

# Introducción

Tailscale utiliza el protocolo [WireGuard](instalar-wireguard-en-proxmox-lxc) para proporcionar conexiones seguras y encriptadas de extremo a extremo. Algunas características clave de Tailscale incluyen:

* **Configuración sin complicaciones**: No requiere configuración manual, lo que permite desplegar una VPN de manera rápida y sencilla.
* **Acceso remoto seguro**: Permite acceder de forma segura a recursos en cualquier infraestructura, ya sea en la nube, en redes privadas virtuales (VPC) o en redes locales.
* **Segmentación de red granular**: Permite segmentar la red para asegurar que los usuarios correctos tengan acceso a los recursos adecuados.
* **Integraciones**: Tailscale se integra con más de 100 herramientas y servicios, lo que facilita su incorporación en cualquier flujo de trabajo.

# Creación del LXC

**Tailscale** se ejecutará sobre un LXC creado desde la _Shell_ del servidor Proxmox contestando a las preguntas de mi _script_ [create_ct.sh](crear-un-lxc-desde-el-cli-de-proxmox):

```
root@pve:~# ./create_ct.sh 
Introduce el CT ID: 311
Introduce el nombre del contenedor: tailscale
Privilegiado (yes/no) [no]: 
¿Utilizarás Docker? (yes/no) [no]: 
Introduce el password del usuario 'root': 
Confirma el password del usuario 'root': 
Introduce el tamaño del disco (GB) [8]: 4
Introduce la memoria RAM (MB) [512]: 
Introduce el número de cores [1]: 
Introduce la dirección IP (192.168.1.x): 192.168.1.95
Introduce los servidores DNS (separados por espacios) [192.168.1.81 192.168.1.82]: 
¿Quieres montar /datos/samba/media en el CT? (yes/no) [no]:
```

Antes de poner en marcha el LXC, hay que añadir las siguientes líneas al fichero `/etc/pve/lxc/311.conf` para poder usar el **dispositivo TUN/TAP** `/dev/net/tun` dentro del contenedor:

```
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```

A continuación, se inicia el LXC mediante el comando `pct start 311` desde la _Shell_ de Proxmox o usando el botón `Start` desde la configuración del propio LXC.

# Acceder al LXC

Se puede acceder al LXC utilizando la opción `Console` de la GUI o, mucho mejor, mediante **SSH** gracias a la configuración de las claves públicas que se hizo en el mismo:

```
ssh root@192.168.1.95
```

Una vez dentro del LXC, hay que: 

* completar el asistente de instalación **TurnKey GNU/Linux - First boot configuration**
  * Pulsar el botón **Skip** en la pantalla _Initialize Hub Services_
  * Pulsar el botón **Skip** en la pantalla _System Notifications and Critical Security Alerts_
  * Pulsar el botón **Install** en la pantalla _Security updates_
  * Pulsar el botón **Advanced Menu**
  * Seleccionar la opción **Quit** para salir de la consola de configuración
  * Pulsar el botón **Yes** para confirmar
* reiniciar el LXC, sobretodo si hay una actualización del _kernel_ (`shutdown -r now`)
* reconfigurar la zona horaria (`dpkg-reconfigure tzdata` y seleccionar `Europe/Madrid`)
* actualizar los paquetes (`apt update` y `apt upgrade -y`)
* desinstalar el paquete `etckeeper` (`apt purge etckeeper`)
* reiniciar el LXC

# Instalación de Tailscale

La instalación de [Tailscale en **Debian** `bookworm`](https://tailscale.com/kb/1174/install-debian-bookworm){:target=_blank} es muy sencilla siguiendo estos pasos:

```
# Añadir la clave oficial GPG de Tailscale
curl -fsSL https://pkgs.tailscale.com/stable/debian/bookworm.noarmor.gpg | \
tee /usr/share/keyrings/tailscale-archive-keyring.gpg >/dev/null

# Añadir el repositorio a las fuentes APT
curl -fsSL https://pkgs.tailscale.com/stable/debian/bookworm.tailscale-keyring.list | \
tee /etc/apt/sources.list.d/tailscale.list >/dev/null

# Actualizar los repositorios e instalar Tailscale
apt update
apt install tailscale -y

# Reiniciar el LXC
shutdown -r now
```

{: .box-note}
Se puede borrar el historial ejecutando el siguiente comando `cat /dev/null > ~/.bash_history && history -c`.

# Configuración de Tailscale

Si se habían añadido las líneas para montar el dispositivo TUN/TAP en el contenedor, el servicio `tailscaled` debería haberse iniciado correctamente y mostrar una _status_ similar al siguiente:

```
root@tailscale ~# systemctl status tailscaled.service 
* tailscaled.service - Tailscale node agent
     Loaded: loaded (/lib/systemd/system/tailscaled.service; enabled; preset: enabled)
     Active: active (running) since Mon 2025-02-24 22:09:22 CET; 5min ago
       Docs: https://tailscale.com/kb/
   Main PID: 111 (tailscaled)
     Status: "Stopped; run 'tailscale up' to log in"
      Tasks: 9 (limit: 18793)
     Memory: 76.6M
        CPU: 367ms
     CGroup: /system.slice/tailscaled.service
             `-111 /usr/sbin/tailscaled --state=/var/lib/tailscale/tailscaled.state --socket=/run/tailscale/tailscaled.sock --port=41641
```

## Habilitar _forwarding_ de IPv4 e IPv6

Para que Tailscale funcione correctamente y permita el tráfico entre los dispositivos conectados, es necesario habilitar el reenvío (_forwarding_) de IPv4 e IPv6 en el sistema.

Para ello se edita el fichero `/etc/sysctl.conf` y se descomentan las siguientes líneas:

```
# Enrutar paquetes IPv4 entre interfaces de red
net.ipv4.ip_forward=1

# Enrutar paquetes IPv6 entre interfaces de red
net.ipv6.conf.all.forwarding=1
```

A continuación hay que recargar la configuración usando el comando `sysctl -p` y, finalmente, reiniciar Tailscale con el comando `systemctl restart tailscaled.service`.

## Crear red Tailscale

Para crear nuestra red Tailscale se utiliza el comando `tailscale up`. Al cabo de unos segundos aparecerá un mensaje en la consola con la URL desde la cual debemos iniciar sesión:

```
To authenticate, visit:

        https://login.tailscale.com/a/xxxxxxxxxxxxx
```

Como Tailscale no mantiene usuarios hay que autenticarse mediante alguno de los proveedores indicados (Google, Microsoft, etc.).

En mi caso utilizaré **GitHub** para autenticarme y añadir el dispositivo tal como se muestra en las siguientes imágenes:

![Login Tailscale][1]

La máquina añadida se debería ver en la sección **Machines** del [Panel de Administración](https://login.tailscale.com/admin/machines){:target=_blank} formando parte de nuestra nueva red Tailscale o **tailnet**:

![Machine][2]

## Acceso a la LAN y nodo de salida

Dos de las configuraciones más habituales de un node en Tailscale son:

* habilitar el [**subnet**](https://tailscale.com/kb/1019/subnets){:target=_blank} para permitir el acceso a la red LAN
* convertirlo en un [**exit node**](https://tailscale.com/kb/1103/exit-nodes){:target=_blank} para permitir la salida a Internet a través de él cuando estamos fuera de la LAN

{: .box-note}
El uso de un nodo de salida es especialmente útil para utilizar servicios sensibles a la dirección IP ;-)

Esto se consigue mediante los siguientes comandos:

```
tailscale down
tailscale up --advertise-routes=192.168.1.0/24 --advertise-exit-node
```

{: .box-note}
Si aparece una advertencia sobre `UDP GRO forwarding` hay que realizar un [cambio en la interfaz `eth0`](https://tailscale.com/s/ethtool-config-udp-gro){:target=_blank}.

En el Panel de Administración, el equipo debería aparecer con los iconos `Subnets` y `Exit Node` tal como muestra la siguiente imagen:

![Subnets/Exit Node][3]

Ahora podemos habilitarlos accediendo al menú lateral `Edit route settings...`, marcando las casillas correspondientes y pulsando el botón `Save`:

![Enable Subnets/Exit Node][4]

## Habilitar UDP GRO _forwarding_

GRO es una técnica de optimización de red que reduce la sobrecarga del procesamiento de paquetes al agrupar múltiples paquetes de red más pequeños en uno más grande.

En Tailscale, si el dispositivo Linux está actuando como **exit node** o **subnet router**, se aconseja habilitar la función de **reenvío de GRO** (Generic Receive Offload) para paquetes UDP recibidos en la interfaz `eth0` y deshabilitar la **lista GRO** de recepción en la misma interfaz:

```
ethtool -K eth0 rx-udp-gro-forwarding on rx-gro-list off
```

Como esta configuración se pierde al reiniciar la máquina, es necesario crear un **servicio** de `systemd` para ejecutar los comandos al inicio.

Para ello únicamente hay que crear un fichero `/etc/systemd/system/ethtool.service` con el siguiente contenido:

```
[Unit]
Description=Configurar ethtool en inicio
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/sbin/ethtool -K eth0 rx-udp-gro-forwarding on
ExecStart=/usr/sbin/ethtool -K eth0 rx-gro-list off
RemainAfterExit=true

[Install]
WantedBy=multi-user.target
```

A continuación se recarga la configuración y se habilita el nuevo servicio:

```
systemctl daemon-reload
systemctl enable ethtool.service
```

Si todo funciona correctamente, al reiniciar este servidor la configuración debería ser la correcta:

```
root@tailscale ~# ethtool -k eth0 | grep rx-udp-gro-forwarding   
rx-udp-gro-forwarding: on

root@tailscale ~# ethtool -k eth0 | grep rx-gro-list
rx-gro-list: off
```

## Deshabilitar _Key expiry_

Para evitar que las claves de este nodo expiren y nos quedemos sin acceso, seleccionar `Disable key expiry` en el menú lateral del dispositivo.

# _Troubleshooting_

Para ver los mensajes de _log_ desde que se puso en marcha el servicio `tailscaled` se puede usar el comando `journalctl` filtrando por servicio y hora de inicio:

```
journalctl -u tailscaled --since="$(systemctl show -p ActiveEnterTimestamp tailscaled.service | \
awk -F'=' '{print $2}')"
```

# Tailscale en Android

La [instalación de Tailscale en Android](https://tailscale.com/kb/1079/install-android){:target=_blank} es muy sencilla. Después de haber descargado la aplicación desde la [Play Store](https://play.google.com/store/apps/details?id=com.tailscale.ipn){:target=_blank}, únicamente hay que autenticarse para registrar el dispositivo en nuestra _tailnet_.

Cuando se activa Tailscale en Android funciona como cualquier otra VPN pero:

* si no se activa el _exit node_, la salida a Internet se realizará a través de la red Wi-Fi o de datos móviles que tengamos en ese momento.
* si se activa el _exit node_, la salida a Internet se realizará a través del nodo que tenemos en casa. Esto hará que la dirección del teléfono sea la dirección IP pública de casa ;-)

{: .box-note}
Se puede comprobar fácilmente nuestra dirección IP con el servicio [2ip.io](https://2ip.io/){:target=_blank}.

[En Android no existe "_VPN on Demand_"](https://github.com/tailscale/tailscale/issues/12086){:target=_blank} como en iOS por lo que habrá que activar o desactivar la VPN de forma manual o utilizar alguna aplicación como [**Tasker**](https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm){:target=_blank} para automatizarlo.

{: .box-note}
Se puede encontrar un ejemplo de este método en el siguiente [Tutorial: Turn Taiscale on/off automatically using Tasker (Android)](https://www.reddit.com/r/Tailscale/comments/141rkyy/tutorial_turn_taiscale_onoff_automatically_using/?rdt=48879){:target=_blank}.

# Usar Pi-hole como DNS

Si se tiene [Pi-hole como servidor DNS](instalar-pihole-en-proxmox-lxc) como es mi caso, se podrían añadir la dirección IP de ese servidor y entonces Tailscale lo utilizará cuando esté activada la VPN.

* Acceder a `DNS - Global nameservers`
* Añadir un `nameserver` de tipo `Custom`: <IP de Pi-hole #1>
* (Opcional) Añadir un `nameserver` de tipo `Custom`: <IP de Pi-hole #2>
* Activar la opción **Override local DNS**

Esto puede ser útil para utilizar las capacidades de bloqueo de anuncios y publicidad de Pi-hole cuando estamos fuera de casa en un dispositivo móvil.

# Referencias

* [Tailscale on a Proxmox host](https://tailscale.com/kb/1133/proxmox){:target=_blank}
* [Tailscale in LXC containers](https://tailscale.com/kb/1130/lxc-unprivileged){:target=_blank}
* [Exit nodes (route all traffic)](https://tailscale.com/kb/1103/exit-nodes){:target=_blank}
* [Proxmox: Running Tailscale](https://medium.com/@rar1871/proxmox-running-tailscale-7929b3eaa31f){:target=_blank}, @Medium
* [Proxmox VE Helper-Scripts: Tailscale](https://community-scripts.github.io/ProxmoxVE/scripts?id=add-tailscale-lxc){:target=_blank}
* [How to create a Tailscale subnet router in an LXC container using Proxmox VE 8.x](https://nihalatwal.com/projects/tailscale-subnet-router-proxmox/){:target=_blank}

### Historial de cambios

* **2025-02-24**: Documento inicial
* **2025-02-27**: Revisión y publicación

[1]: /assets/img/blog/2025-02-24_image_1.png "Login Tailscale"
[2]: /assets/img/blog/2025-02-24_image_2.png "Machine"
[3]: /assets/img/blog/2025-02-24_image_3.png "Subnets/Exit Node"
[4]: /assets/img/blog/2025-02-24_image_4.png "Enable Subnets/Exit Node"
