---
layout : post
blog-width: true
title: 'Docker en Proxmox LXC con TurnKey Core'
date: '2023-10-21 14:55:00'
last-updated: '2025-02-13 19:25:40'
published: true
tags:
- Proxmox
- Docker
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2023-10-21_cover.png"
thumbnail-img: ""  
---

# Introducción

Este artículo explica cómo instalar [**Docker Engine**](https://www.docker.com/products/container-runtime/) en un [**contenedor Linux (LXC)**](https://linuxcontainers.org/lxc/introduction/) que se ejecutará en el servidor [Proxmox VE 8.0.2](proxmox-ve-802-en-un-dell-optiplex-7050) que instalé hace unos meses.

{: .box-note}
**Nota**: El soporte oficial de Proxmox recomienda ejecutar Docker en máquinas virtuales, pero la desventaja es que éstas requieren más recursos del hipervisor. Un contenedor Linux (LXC) permite ejecutar Docker con una fracción de los requisitos de recursos y velocidades de arranque mucho más rápidas.

# Actualizar Proxmox

Antes de empezar, nos aseguraremos que Proxmox esté correctamente actualizado. Se puede comprobar la versión actual mediante el comando `pveversion`:

```
root@pve:~# pveversion 
pve-manager/8.0.3/bbf3993334bfa916 (running kernel: 6.2.16-3-pve)
```

A continuación se procede a realizar la actualización usando el habitual comando `apt`:

```
# Actualizar repositorios
apt update

# Actualizar paquetes
apt upgrade -y

# Actualizar paquetes y borrar dependencias no necesarias
# apt full-upgrade -y

# Reiniciar para aplicar los cambios
reboot -h now
```

Si todo funciona correctamente, Proxmox habrá actualizado sus paquetes (entre ellos el _kernel_ y la interfaz web):

```
root@pve:~# pveversion 
pve-manager/8.0.4/d258a813cfa6b390 (running kernel: 6.2.16-15-pve)
```

# Descargar plantilla de TurnKey Core

Antes de poder crear un contenedor Linux es necesario descargar la plantilla del SO que ejecutaremos en él. En este caso se utilizará [TurnKey Core](https://www.turnkeylinux.org/core), una pequeña distribución basada en **Debian GNU/Linux 10**

* Expandir el centro de datos (_Datacenter_)
* Expandir el servidor (_node_): `pve`
* Seleccionar el almacenamiento (_storage_): `local`
* Seleccionar la opción **CT Templates** del menú lateral
* Pulsar el botón **Templates** y buscar `turnkey-core`

> Si no aparece, se puede actualizar la base de datos de templates usando el comando `pveam update` desde la _shell_ de Proxmox.

* Seleccionar el paquete `turnkey-core`
* Pulsar el botón **Download**
* Esperar a que finalice la descarga (`TASK OK`) y cerrar la ventana

{: .box-note}
**Nota**: A día de hoy, 21 de octubre de 2023, este proceso descarga el fichero [`debian-11-turnkey-core_17.1-1_amd64.tar.gz`](http://mirror.turnkeylinux.org/turnkeylinux/images/proxmox/debian-11-turnkey-core_17.1-1_amd64.tar.gz) correspondiente a la versión [17.1](https://www.turnkeylinux.org/updates/new-turnkey-core-version-171) aunque en la web de TurnKey ya esté disponible la versión [18.0](https://www.turnkeylinux.org/updates/new-turnkey-core-version-180).

# Contenedor Linux (CT)

## Instalación

A continuación se puede comenzar el proceso para crear el contenedor Linux (**CT** en Proxmox) siguiendo estos pasos:

* Pulsar el botón `Create CT` en la parte superior derecha de la pantalla
* **General**
  * Node: pve
  * CT ID: `300` (se usará este rango para contenedores Linux)
  * Hostname: `docker` (un nombre descriptivo del uso que se dará al contenedor)
  * Unprivileged container: Sí
  * Nesting: Sí
  * Password: `********` (para acceder al CLI)
  * Confirm password: `********`

{: .box-note}
**Nota**: Es más recomandable cargar una clave pública, que permita acceder por SSH sin necesitar usuario y contraseña, mediante la opción **Load SSH Key File**.

* **Template**
  * Storage: `local`
  * Template: `debian-11-turnkey_core_17.1-1_amd64.tar.gz`
* **Disks**
  * rootfs
    * Storage: `local-lvm`
    * Disk size (GiB): `8`
* **CPU**
  * Cores: 1
* **Memory**
  * Memory (MiB): `1024`
  * Swap (MiB): 512
* **Network**
  * Name: eth0
  * Bridge: vmbr0
  * Firewall: Sí
  * IPv4: `DHCP`
  * IPv6: `DHCP`
* **DNS**
  * DNS domain: use host settings
  * DNS servers: use host settings
* **Confirm**  
  * No marcar _Start after created_
* Pulsar el botón **Finish**
* Esperar a que finalice la creación del CT (`TASK OK`) y cerrar la ventana

## Configuración

Para poder ejecutar Docker dentro de un LXC es necesario tener habilitadas las opciones [**_Nesting_**](https://pve.proxmox.com/wiki/Nested_Virtualization) y **keyctl**:

* Expandir el centro de datos (_Datacenter_)
* Expandir el servidor (_node_): `pve`
* Seleccionar el contenedor (_container_): `300 (dockers)`
* Seleccionar la opción **Options**
* Editar los siguientes valores:
  * **Features**:
    * Marcar la opción **keyctl**
    * Marcar la opción **Nesting**
  * **Start at boot**
    * Marcar la opción **Start at boot**

## Ejecución

A continuación se pone en marcha el contenedor:

* Expandir el centro de datos (_Datacenter_)
* Expandir el servidor (_node_): `pve`
* Seleccionar el contenedor (_container_): `300 (dockers)`
* Pulsar el botón **Start**
* Pulsar el botón **Console** para ver la instalación

Una vez iniciado el contenedor Linux, se puede iniciar sesión:

* dockers login: `root`
* Password: `********`

Cuando aparezca el asistente de instalación **First Boot Configuration**:

* Pulsar el botón **Skip** en la pantalla _Initialize Hub Services_
* Pulsar el botón **Skip** en la pantalla _System Notifications and Critical Security Alerts_
* Pulsar el botón **Install** en la pantalla _Security updates_

{: .box-note}
Si aparece el mensaje `A security update to the kernel requires a reboot to go into effect` se reiniciará el LXC despues de salir del asistente.

Al finalizar la instalación, en la _TurnKey GNU/Linux Configuration Console_ se mostrarán las URLs de servicio, por ejemplo:

```
Web shell: https://192.168.1.241:12320
Webmin: https://192.168.1.241:12321
SSH/SFTP: root@192.168.1.241 (port 22)
```

* Pulsar el botón **Advanced Menu**
* Seleccionar la opción **Quit** para salir de la consola de configuración
* Pulsar el botón **Yes** para confirmar

## Actualización

{: .box-note}
**Nota**: Aunque no es estrictamente necesario, se recomienda cambiar la zona horaria antes de seguir. Para ello se puede utilizar el comando `dpkg-reconfigure tzdata` y seleccionar `Europe/Madrid`.

Se actualizan los repositorios de la distribución Debian usando el comando `apt update`:

```
root@docker ~# apt update
* Unprivileged container [x]
* Nesting [x]
Hit:1 http://deb.debian.org/debian bullseye InRelease
Hit:2 http://security.debian.org bullseye-security InRelease
* 
Ign:3 http://archive.turnkeylinux.org/debian bullseye-security InRelease
Ign:4 http://archive.turnkeylinux.org/debian bullseye InRelease
Hit:5 http://archive.turnkeylinux.org/debian bullseye-security Release
Hit:7 http://archive.turnkeylinux.org/debian bullseye Release
Reading package lists... Done                             
Building dependency tree... Done
Reading state information... Done
63 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

A continuación se actualizan los paquetes usando el comando `apt upgrade -y`:

```
root@docker ~# apt upgrade -y
* Unprivileged container [x]
* Nesting [x]
Reading package lists... Done
Building dependency tree... Done
* 
Reading state information... Done
Calculating upgrade... Done
The following packages will be upgraded:
  adduser base-files bash cpio debian-archive-keyring distro-info-data dpkg grep inithooks libbsd0 libgssapi-krb5-2
  libk5crypto3 libkrb5-3 libkrb5support0 libncurses6 libncursesw6 libpcre2-8-0 libpython2.7-minimal libpython2.7-stdlib
  libssl1.1 libsystemd0 libtasn1-6 libtinfo6 libudev1 logrotate nano ncurses-base ncurses-bin ncurses-term openssh-client
  openssh-server openssh-sftp-server openssl postfix python2.7 python2.7-minimal qemu-guest-agent ssh systemd systemd-sysv
  traceroute tzdata udev webmin webmin-authentic-theme webmin-custom webmin-fail2ban webmin-fdisk webmin-filemin
  webmin-firewall webmin-firewall6 webmin-lvm webmin-mount webmin-net webmin-passwd webmin-postfix webmin-raid webmin-shell
  webmin-software webmin-sshd webmin-syslog webmin-updown webmin-useradmin
63 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
Need to get 44.1 MB of archives.
After this operation, 406 kB of additional disk space will be used.
Get:1 http://deb.debian.org/debian bullseye/main amd64 base-files amd64 11.1+deb11u8 [70.2 kB]
Get:2 http://deb.debian.org/debian bullseye/main amd64 bash amd64 5.1-2+deb11u1 [1417 kB] 
[...]
Processing triggers for mailcap (3.69) ...
Processing triggers for initramfs-tools (0.140) ...
[master 5fe3453] committing changes in /etc made by "apt upgrade -y"
 32 files changed, 122 insertions(+), 40 deletions(-)
 create mode 100644 apt/trusted.gpg.d/debian-archive-bookworm-automatic.gpg
 create mode 100644 apt/trusted.gpg.d/debian-archive-bookworm-security-automatic.gpg
 create mode 100644 apt/trusted.gpg.d/debian-archive-bookworm-stable.gpg
 delete mode 100644 apt/trusted.gpg.d/debian-archive-stretch-automatic.gpg
 delete mode 100644 apt/trusted.gpg.d/debian-archive-stretch-security-automatic.gpg
 delete mode 100644 apt/trusted.gpg.d/debian-archive-stretch-stable.gpg
 create mode 100755 webmin/.post-install
 create mode 100755 webmin/.pre-install
 create mode 100755 webmin/.reload-init
 create mode 100755 webmin/.reload-init-systemd
 create mode 100755 webmin/.restart-by-force-kill-init
 create mode 100755 webmin/.restart-init
 create mode 100755 webmin/.start-init
 create mode 100755 webmin/.stop-init
 create mode 100644 webmin/installed.cache
 create mode 100755 webmin/restart-by-force-kill
Enumerating objects: 1222, done.
Counting objects: 100% (1222/1222), done.
Delta compression using up to 2 threads
Compressing objects: 100% (773/773), done.
Writing objects: 100% (1222/1222), done.
Total 1222 (delta 77), reused 1158 (delta 55), pack-reused 0
```

* En la pantalla _Postfix Configuration_, seleccionar la opción **No configuration** y pulsar el botón **OK**
* En la pantalla _Configuring openssh-server_, seleccionar la opción **Keep the local version currently installed** para conservar el fichero `sshd_config` y pulsar el botón **OK**

{: .box-note}
**Nota**: En este momento es recomendable reiniciar el LXC usando el comando `reboot` (sobretodo si se han actualizado muchos paquetes).

# Docker Engine

## Instalación

La [instalación de **Docker Engine**](https://docs.docker.com/engine/install/debian/#install-using-the-repository) se realiza desde el repositorio `apt` oficial ejecutando los siguientes comandos:

```
# Add Docker's official GPG key:
apt update
apt install ca-certificates curl gnupg
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
chmod a+r /etc/apt/keyrings/docker.gpg

# Add the repository to Apt sources:
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  tee /etc/apt/sources.list.d/docker.list > /dev/null
apt update

# Install latest Docker packages
apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## Comprobar servicio `docker.service`

Para comprobar que Docker se ha instalado correctamente en el LXC, se puede ejecutar el comando `systemctl status docker` que muestra información acerca del servicio:

```
root@docker ~# systemctl status docker
* docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2023-10-21 15:01:43 UTC; 4min 9s ago
TriggeredBy: * docker.socket
       Docs: https://docs.docker.com
   Main PID: 15121 (dockerd)
      Tasks: 10
     Memory: 30.0M
        CPU: 290ms
     CGroup: /system.slice/docker.service
             `-15121 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

Oct 21 15:01:43 dockers systemd[1]: Starting Docker Application Container Engine...
Oct 21 15:01:43 dockers dockerd[15121]: time="2023-10-21T15:01:43.561642291Z" level=info msg="Starting up"
Oct 21 15:01:43 dockers dockerd[15121]: time="2023-10-21T15:01:43.629436598Z" level=info msg="Loading containers: start."
Oct 21 15:01:43 dockers dockerd[15121]: time="2023-10-21T15:01:43.781346558Z" level=info msg="Loading containers: done."
Oct 21 15:01:43 dockers dockerd[15121]: time="2023-10-21T15:01:43.804246801Z" level=info msg="Docker daemon" commit=1a79695 graphdriver=overlay2 version=24.0.6
Oct 21 15:01:43 dockers dockerd[15121]: time="2023-10-21T15:01:43.804344432Z" level=info msg="Daemon has completed initialization"
Oct 21 15:01:43 dockers dockerd[15121]: time="2023-10-21T15:01:43.857500717Z" level=info msg="API listen on /run/docker.sock"
Oct 21 15:01:43 dockers systemd[1]: Started Docker Application Container Engine.
Oct 21 15:05:14 dockers dockerd[15121]: time="2023-10-21T15:05:14.689826171Z" level=info msg="ignoring event" container=e8fd306470a1d49d66d5f9123e32d7f1031f94f94578af905f9a8ea7fa1bd374 module=libcontainerd namespace=moby topic=/tasks/delete type="*events.TaskDelete"
```

## Ejecutar aplicación `hello-world`

Para comprobar que todo funciona correctamente, se puede ejecutar el comando `docker run hello-world`:

```
root@docker ~# docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
719385e32844: Pull complete 
Digest: sha256:88ec0acaa3ec199d3b7eaf73588f4518c25f9d34f58ce9a0df68429c5af48e8d
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

# Instalar **Portainer** (opcional)

Instalar [Portainer]() es una opción muy recomendable para poder administrar los contenedores Docker de una forma más sencilla.

Ya expliqué cómo [instalar Portainer CE en Docker](instalacion-de-portainer-ce-en-docker) de forma detallada por lo que aquí añadiré únicamente una forma rápida de hacerlo:

```
# Crear los directorios para el Docker y su volumen persistente
mkdir -p ~/dockers/portainer-ce
mkdir -p ~/volumes/portainer-ce

# Crear el fichero docker-compose.yml
cd dockers/portainer-ce
cat > docker-compose.yml << EOF
services:
  portainer-ce:
    image: portainer/portainer-ce:latest
    container_name: portainer-ce
    volumes:
    - /etc/localtime:/etc/localtime:ro
    - /var/run/docker.sock:/var/run/docker.sock
    - ~/volumes/portainer-ce:/data
    ports:
    - 9443:9443
    restart: always
    security_opt:
      - no-new-privileges:true    
EOF

# Poner en marcha el Docker
docker compose up -d
```

Al ejecutar el comando `docker compose up -d` se descargará la última imagen de Portainer-CE y se ejecutará tal como se muestra a continuación:

```
root@docker ~/dockers/portainer-ce# docker compose up -d
[+] Running 12/12
 ✔ portainer-ce 11 layers [⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿]      0B/0B      Pulled                                                 6.9s
   ✔ 795a208431d7 Pull complete                                                                                    0.5s
   ✔ 4f272ca3dde3 Pull complete                                                                                    0.5s
   ✔ 5171176db7f2 Pull complete                                                                                    0.8s
   ✔ 52e9438966a5 Pull complete                                                                                    1.7s
   ✔ 43d4775415ac Pull complete                                                                                    2.0s
   ✔ c1cad9f5200f Pull complete                                                                                    2.0s
   ✔ 27d6dca9cab4 Pull complete                                                                                    2.4s
   ✔ 231d7e50ef35 Pull complete                                                                                    3.8s
   ✔ 589f2af34593 Pull complete                                                                                    3.0s
   ✔ 5fc2ddaa6f07 Pull complete                                                                                    3.5s
   ✔ 4f4fb700ef54 Pull complete                                                                                    3.9s
[+] Running 2/2
 ✔ Network portainer-ce_default  Created                                                                           0.1s
 ✔ Container portainer-ce        Started
```

Finalmente se podría acceder a `https://<ip-lxc>:9443` para realizar una configuración inicial básica y comenzar a utilizarlo.

# Referencias

* [Docker in Proxmox 7 LXC with Turnkey Core - Lower Resources by 80% Compared to VMs](https://www.youtube.com/watch?v=gXuLiglJceY=) (YouTube)
* [Setup and Install Docker in a Proxmox 7 LXC Container](https://thehomelab.wiki/books/promox-ve/page/setup-and-install-docker-in-a-promox-7-lxc-conainer)
* [Running Docker containers in Proxmox containers](https://forum.proxmox.com/threads/running-docker-containers-in-proxmox-containers.81660/)

A continuación se indican algunas referencias que van más allá al usar ZFS, FUSE-OverlayFS, etc.

* [Running docker inside an unprivileged LXC container on Proxmox](https://du.nkel.dev/blog/2021-03-25_proxmox_docker/)
* [Proxmox: Run Docker on Linux Containers (LXC)](https://benheater.com/proxmox-run-docker-on-linux-containers-lxc/)
* [Running Docker in LXC With Proxmox 7.1](https://www.weisb.net/running-docker-in-lxc-with-proxmox-7-1/)
* [FUSE-OverlayFS @ Reddit](https://www.reddit.com/r/Proxmox/comments/va76fz/fuse_overlayfs/)
* [Docker LXC Unprivileged container on Proxmox 7 with ZFS](https://forum.proxmox.com/threads/docker-lxc-unprivileged-container-on-proxmox-7-with-zfs.99796/)

### Historial de cambios

* **2023-10-21**: Documento inicial
* **2025-02-15**: Reinicio del LXC si hay actualización del kernel
