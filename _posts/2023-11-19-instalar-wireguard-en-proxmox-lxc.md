---
layout : post
blog-width: true
title: 'Instalar Wireguard en Proxmox LXC'
date: '2023-11-19 16:26:11'
published: true
tags:
- Proxmox
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2023-11-19_cover.png"
thumbnail-img: ""
---

Para seguir el proceso de migración de mi _Home Lab_ desde una pequeña [Raspberry Pi 4](instalar-raspberry-pi-os-64bit) a un [OptiPlex 7050 ejecutando Proxmox](proxmox-ve-802-en-un-dell-optiplex-7050) ahora le toca el turno a [**Wireguard**](https://www.wireguard.com/){:target="_blank"}, una VPN extremadamente simple pero rápida y moderna que utiliza [criptografía](https://www.wireguard.com/protocol/){:target="_blank"} de última generación.

En este ocasion, en lugar de realizar la [instalación de Wireguard en Docker](instalacion-de-wireguard-en-docker), se utilizará un contenedor ligero LXC de Proxmox con TurnKey Core. El procedimiento para crear este LXC es el mismo que usé para [ejecutar Docker en Proxmox LXC](docker-en-proxmox-lxc-con-turnkey-core).

# Características del LXC

Esta **VPN** con [Wireguard](https://www.wireguard.com/){:target="_blank"} se creará sobre un LXC con las siguientes características:

* LXC no privilegiado, **con** _nesting_ y sin `keyctl`
* CT ID: **303**
* Hostname: **wireguard**
* Password + SSH key file
* Disco: 2GB en `local_lvm`
* CPU: 1 _core_
* Memoria: 512MB
* IP estática: **192.168.1.83/24**
* Gateway: **192.168.1.1**
* Dominio: **home**
* DNS: **192.168.1.81,192.168.1.82**

A continuación se pone en marcha el LXC desde la CLI de Proxmox y se procede a configurarlo de la misma manera que ya expliqué en el artículo sobre [LXC y Docker](docker-en-proxmox-lxc-con-turnkey-core):

```
pct start 303
```

# Acceder al LXC

Se puede acceder al LXC utilizando la opción `Console` de la GUI o, mucho mejor, mediante **SSH** gracias a la configuración de las claves públicas que se hizo en el mismo:

```Bash
ssh root@192.168.1.83
```

A continuación se realiza la configuración habitual:

* First Boot Configuration: `Skip` / `Skip` / `Install`
* Advanced Menu: `Quit`
* Zona horaria **Europe/Madrid** con `dpkg-reconfigure tzdata`
* Actualizar con `apt update && apt upgrade -y`
* Reiniciar con `reboot`

# Instalación de PiVPN

La instalación de Wireguard usando [PiVPN](https://pivpn.io/){:target=_blank} es muy sencilla usando el script `install.sh`:

```
cd ~
wget -O install.sh https://install.pivpn.io
bash install.sh
```

Si se cumplen los requisitos para instalar Wireguard, se ejecutará el asistente _PiVPN Automated Installer_ mediante el cual se proporcionará la información necesaria para la instalación:

* Force routing to block [IPv6 leak](https://ipv6leak.com/){:target=_blank}: **No**
* Local user: `wireguard`
* Password: `********`
* Choose VPN: **Wireguard**

Al finalizar la instalación, si no hay ningún problema, continuarán las preguntas del asistente:

* Default Wireguard port: **51821**
* DNS Provider: `Custom`
* Upstream DNS providers: **192.168.1.81,192.168.1.82**
* Public IP or DNS: `DNS Entry`
* Public DNS: **wireguard.myhome.com**
* Unattended upgrades: **Yes**
* Reboot: **Yes**

{: .box-note}
**Nota**: Se puede utilizar el comando `pivpn` para añadir clientes, depurar problemas, generar códigos QR, etc. Pero antes es necesario modificar el fichero `/etc/pivpn/wireguard/setupVars.conf` para cambiar las redes que se enrutarán a través de la VPN.

```
# Originales
ALLOWED_IPS="0.0.0.0/0"
# Nuevas
ALLOWED_IPS="192.168.1.0/24"
```

# Añadir clientes

Para añadir un cliente se utiliza el comando `pivpn -a -n client1`:

```
root@wireguard ~# pivpn -a -n client1
::: Client Keys generated
::: Client config generated
::: Updated server config
::: WireGuard reloaded
======================================================================
::: Done! manel.conf successfully created!
::: client1.conf was copied to /home/wireguard/configs for easytransfer.
::: Please use this profile only on one device and create additional
::: profiles for other devices. You can also use pivpn -qr
::: to generate a QR Code you can scan with the mobile app.
======================================================================
```

A continuación se usa el comando `pivpn -qrcode client1` para mostrar el código QR que permite añadir el perfil en un cliente móvil de forma fácil:

![Código QR Wireguard][1]

# Referencias

* [Getting PiVPN to run on Proxmox LXC container](https://www.vanpolen.biz/posts/pivpn-on-lxc-container-proxmox/){:target=_blank}
* [Set up Wireguard using PiVPN inside LXC](https://nocin.eu/wireguard-set-up-wireguard-using-pivpn-inside-lxc/){:target=_blank}
* [OpenVPN in LXC](https://pve.proxmox.com/wiki/OpenVPN_in_LXC){:target=_blank} @ Proxmox Wiki 
* [WGDashboard](https://github.com/donaldzou/WGDashboard){:target=_blank}, usada por _Proxmox VE Helper Scripts_
* [Wg Gen Web](https://github.com/vx3r/wg-gen-web){:target=_blank}, para ejecutar en Docker
* [Wireguard-UI](https://github.com/ngoduykhanh/wireguard-ui){:target=_blank}, interfaz realizada en Go
* [Generating WireGuard QR codes for fast mobile deployments](https://serversideup.net/generating-wireguard-qr-codes-for-fast-mobile-deployments/){:target=_blank}
* [How to monitor who's connected to your Wireguard VPN](https://www.procustodibus.com/blog/2021/01/how-to-monitor-wireguard-activity/){:target=_blank}

[1]: /assets/img/blog/2023-11-19_image_1.png "Código QR Wireguard"
