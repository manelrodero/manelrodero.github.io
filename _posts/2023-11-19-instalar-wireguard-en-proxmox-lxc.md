---
layout : post
blog-width: true
title: 'Instalar WireGuard en Proxmox LXC'
date: '2023-11-19 16:26:11'
last-updated: '2024-02-12 20:35:29'
published: true
tags:
- HomeLab
- Proxmox
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2023-11-19_cover.png"
thumbnail-img: ""
---

Para seguir el proceso de migración de mi _Home Lab_ desde una pequeña [Raspberry Pi 4](instalar-raspberry-pi-os-64bits) a un [OptiPlex 7050 ejecutando Proxmox](proxmox-ve-802-en-un-dell-optiplex-7050) ahora le toca el turno a [**WireGuard**](https://www.wireguard.com/){:target="_blank"}, una VPN extremadamente simple pero rápida y moderna que utiliza [criptografía](https://www.wireguard.com/protocol/){:target="_blank"} de última generación.

En esta ocasion, en lugar de realizar la [instalación de WireGuard en Docker](instalacion-de-wireguard-en-docker), se utilizará un contenedor ligero LXC de Proxmox con TurnKey Core. El procedimiento para crear este LXC es el mismo que usé para [ejecutar Docker en Proxmox LXC](docker-en-proxmox-lxc-con-turnkey-core).

# Características del LXC

Esta **VPN** con [WireGuard](https://www.wireguard.com/){:target="_blank"} se ejecutará sobre un LXC con las siguientes características:

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
* DNS: **192.168.1.81 192.168.1.82**

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

La instalación de WireGuard usando [PiVPN](https://pivpn.io/){:target="_blank"} es muy sencilla usando el script `install.sh`:

```
cd ~
wget -O install.sh https://install.pivpn.io
bash install.sh
```

Si se cumplen los requisitos para instalar WireGuard, se ejecutará el asistente _PiVPN Automated Installer_ mediante el cual se proporcionará la información necesaria para la instalación:

* Force routing to block [IPv6 leak](https://ipv6leak.com/){:target="_blank"}: **No**
* Local user: `wireguard`
* Password: `********`
* Choose VPN: **WireGuard**

Al finalizar la instalación, si no hay ningún problema, continuarán las preguntas del asistente:

* Default WireGuard port: **51821**
* DNS Provider: `Custom`
* Upstream DNS providers: **9.9.9.9, 149.112.112.112**
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

A continuación se usa el comando `pivpn -qr client1` para mostrar el código QR que permite añadir el perfil en un cliente móvil de forma fácil:

![Código QR WireGuard][1]

# Actualización de WireGuard

La actualización de PiVPN se realizaba usando el comando `pivpn -up` pero, desde el año 2020, los componentes de la VPN (WireGuard) se actualizan mediante:

```
apt update && apt upgrade -y
```

# DNS privado en Android

Android 9+ admite DNS sobre TLS para proteger las consultas mediante cifrado. Esta opción, llamada [**DNS privado**](https://dnsprivacy.org/public_resolvers/){:target="_blank"}, se encuentra en `Ajustes` &rarr; `Redes e Internet` &rarr; `Avanzado` &rarr; `DNS privado`.

Para configurar esta opción se selecciona la opción `Nombre de host del proveedor de DNS privado` y se introduce el proveedor que se desee utilizar, por ejemplo:

* Quad9:
  * Malware Blocking, DNSSEC Validation: `dns.quad9.net`
  * Malware blocking, DNSSEC Validation, ECS enabled: `dns11.quad9.net`
* Cloudflare:
  * DNS Resolver: `one.one.one.one`
  * Block malware for Families: `security.cloudflare-dns.com`
  * Block malware and adult content for Families: `family.cloudflare-dns.com`

{: .box-note}
A diferencia de las versiones anteriores de Android, este método también garantiza que no sea necesario configurar un DNS propio para cada nueva red Wi-Fi a la que se una el teléfono inteligente.

En algunos teléfonos, por ejemplo [en mi Pixel 7a con GrapheneOS](uso-de-grapheneos-con-google-play-services), al configurar esta opción y activar WireGuard se pierde la conexión a Internet. Esto es debido al hecho de no poder enrutar las peticiones DNS a través de la VPN por una configuración incorrecta de la misma.

Para evitar este problema, únicamente hay que añadir la dirección o direcciones IP del DNS privado a la lista de direcciones permitidas en el fichero `/etc/pivpn/wireguard/setupVars.conf`:

```
ALLOWED_IPS="192.168.1.0/24, 9.9.9.9/32, 149.112.112.112/32"
```

# Referencias

* [Getting PiVPN to run on Proxmox LXC container](https://www.vanpolen.biz/posts/pivpn-on-lxc-container-proxmox/){:target="_blank"}
* [Set up WireGuard using PiVPN inside LXC](https://nocin.eu/wireguard-set-up-wireguard-using-pivpn-inside-lxc/){:target="_blank"}
* [OpenVPN in LXC](https://pve.proxmox.com/wiki/OpenVPN_in_LXC){:target="_blank"} @ Proxmox Wiki
* [WireGuard VPN: Instalación y configuración de la mejor VPN](https://www.redeszone.net/tutoriales/vpn/wireguard-vpn-configuracion/){:target="_blank"} @ RedesZone.net
* [WGDashboard](https://github.com/donaldzou/WGDashboard){:target="_blank"}, usada por _Proxmox VE Helper Scripts_
* [Wg Gen Web](https://github.com/vx3r/wg-gen-web){:target="_blank"}, para ejecutar en Docker
* [WireGuard-UI](https://github.com/ngoduykhanh/wireguard-ui){:target="_blank"}, interfaz realizada en Go
* [How to monitor who's connected to your WireGuard VPN](https://www.procustodibus.com/blog/2021/01/how-to-monitor-wireguard-activity/){:target="_blank"}
* [WireGuard-Manager](https://github.com/complexorganizations/wireguard-manager){:target="_blank"}
* [Wireguard Config Generator](https://www.wireguardconfig.com/){:target="_blank"}
* [Chris Swanda WireGuard Setup](https://gist.github.com/chrisswanda/88ade75fc463dcf964c6411d1e9b20f4){:target="_blank"}
* [Pro Custodibus](https://www.procustodibus.com/){:target="_blank"}

### Historial de cambios

* **2023-11-19**: Artículo original
* **2023-12-11**: Cambio de los DNS públicos a [Quad9](https://www.quad9.net/service/service-addresses-and-features/){:target="_blank"}
* **2024-02-12**: Cambio en Allowed IPs para soportar DNS privado en Android

[1]: /assets/img/blog/2023-11-19_image_1.png "Código QR Wireguard"
