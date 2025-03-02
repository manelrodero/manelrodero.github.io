---
layout : post
blog-width: true
title: 'Instalar Prowlarr en Proxmox LXC'
date: '2024-10-05 21:38:01'
#last-updated: '2024-10-05 21:38:01'
published: true
tags:
- Proxmox
author:
  display_name: Manel Rodero
#cover-img: "/assets/img/blog/2024-10-05_cover.png"
thumbnail-img: ""
---

Para seguir el proceso de migración de mi _Home Lab_ desde una pequeña [Raspberry Pi 4](instalar-raspberry-pi-os-64bits) a un [OptiPlex 7050 ejecutando Proxmox](proxmox-ve-802-en-un-dell-optiplex-7050) ahora le toca el turno a [**Prowlarr**](https://github.com/Prowlarr/Prowlarr){:target="_blank"}, una aplicación que funciona como un **servidor proxy** entre las aplicaciones (Sonarr, Radarr, Lidarr, Readarr) y los _trackers_ Torrent o los _indexer_ Usenet.

En esta ocasion, en lugar de realizar la [instalación de Prowlarr en Docker](instalacion-de-prowlarr-en-docker), se utilizará un contenedor ligero LXC de Proxmox con Debian 12.

# Creación del LXC

[Prowlarr](https://prowlarr.com/){:target="_blank"} se ejecutará sobre un LXC creado desde la _Shell_ del servidor Proxmox mediante los [Proxmox VE Helper-Scripts](https://tteck.github.io/Proxmox/){:target="_blank"}:

```bash
mkdir -p ~/tteck && cd ~/tteck
wget -nv https://github.com/tteck/Proxmox/raw/main/ct/prowlarr.sh
# Revisar el script ;-)
bash prowlarr.sh
```

Al ejecutar el _script_ anterior, aparecerá el asistente de instalación:

* This will create a New Prowlarr LXC. Proceed? `Yes`
* Use Default Settings? `Advanced`
* To make a selection, use the Spacebar. `Ok`
* If the default Linux distribution is not adhered to, script support will be discontinued (debian 12). `Ok`
* Choose Distribution: **debian**
* Choose version: **12 Bookworm**
* Choose Type: **1 Unprivileged**
* Set Root Password (needed for root ssh access): **xxxxxxxx**
* Verify Root Password: **xxxxxxxx**
* Set Container ID: **311**
* Set Hostname: **prowlarr**
* Set Disk Size in GB: **4**
* Allocate CPU Cores: **2**
* Allocate RAM in MiB: **1024**
* Set a Bridge: **vmbr0**
* Set a Static IPv4 CIDR Address (/24): **192.168.1.88/24**
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
* Ready to create Prowlarr LXC? `Yes`

{: .box-note}
Deshabilitar IPv6 implica añadir la siguiente configuración al LXC: `net.ipv6.conf.all.disable_ipv6 = 1`

Antes de poner en marcha el LXC, se pueden deshabilitar un par de opciones que no son necesarias para el funcionamiento de Prowlarr y que han sido añadidas por el _script_ ejecutado anteriormente:

* Expandir el centro de datos (_Datacenter_)
* Expandir el servidor (_node_): `pve`
* Seleccionar el contenedor (_container_): `311 (prowlarr)`
* Seleccionar la opción **Options**
* Editar los siguientes valores:
  * **Features**:
    * Desmarcar la opción **keyctl**
    * Desmarcar la opción **Nesting**

{: .box-note}
Al deshabilitar _nesting_ en un [LXC Debian 12](https://pve.proxmox.com/pve-docs/pct.conf.5.html){:target="_blank"}, el acceso a la _Console_ del LXC es unos segundos más lento.

A continuación, se reinicia el LXC mediante el comando `pct reboot 311` desde la _Shell_ de Proxmox o usando el botón `Reboot` desde la configuración del propio LXC.

# Configuración del LXC

Al no haber habilitado el acceso para el usuario `root` mediante SSH, se puede acceder al LXC utilizando el botón `Console` desde la interfaz gráfica del propio LXC.

De todos modos, la configuración inicial de Prowlarr se realiza mediante su interfaz web disponible en el puerto **9696** de la IP estática configurada anteriormente ([http://192.168.1.88:9696](http://192.168.1.88:9696){:target="_blank"}):

Lo primero y más importante es activar el login y crear una contraseña segura para el usuario administrador:

* Settings > General > Authentication > Forms (Login page)
* Settings > General > Username > `admin`
* Settings > General > Password > `*********`
* Settings > General > Analytics > Disable Send Anonymous Usage Data

A continuación se graban los cambios y se reinicia la aplicación cuando ésta lo indique.

Para que la aplicación pueda descargar ficheros, se debe configurar un cliente Torrent o Usenet:

* Settings > Download clients > Add
  * Torrent > qBittorrent
  * Name: qBittorrent
  * Host: 192.168.1.91
  * Port: 8080
  * Username: `admin`
  * Password: `**********`
  * Category: prowlarr
  * Test > Save

A continuación se puede añadir uno o más **Indexer** (públicos o privados) pulsado el botón `+Add indexer` en la sección `Indexers` del menú lateral.

Estos indexer se pueden añadir a Sonarr, Radarr, Lidarr, etc. de forma automática desde Prowlarr añadiendo las aplicaciones correspondientes desde el menú `Settings` &rarr; `Apps`.

Por ejemplo, para añadir **Radarr** ser haría lo siguiente:

* Settings > Apps > Add
  * Name: Radarr
  * Sync Level: Add and Remove Only
  * Prowlarr Server: `http://192.168.1.88:9696`
  * Radarr Server: `http://192.168.1.89:7878`
  * ApiKey (copiarla desde `Settings` &rarr; `General` de Radarr)
  * Test > Save

# Referencias

* [Prowlarr](https://prowlarr.com/){:target="_blank"}
* [Prowlarr GitHub](https://github.com/Prowlarr/Prowlarr){:target="_blank"}
* [Guía de Prowlarr: Cómo instalarlo, configurarlo y utilizarlo](https://www.rapidseedbox.com/es/blog/prowlarr-guide){:target="_blank"}
* [Universo -arr Parte II: Prowlarr](https://elblogdelazaro.org/posts/2023-01-02-universo-arr-parte-ii-prowlarr/){:target="_blank"}

### Historial de cambios

* **2024-10-05**: Documento inicial
