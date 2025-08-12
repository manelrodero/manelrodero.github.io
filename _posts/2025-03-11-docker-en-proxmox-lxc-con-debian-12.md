---
layout : post
blog-width: true
title: 'Docker en Proxmox LXC con Debian 12'
date: '2025-03-11 20:08:57'
#last-updated: '2025-03-11 20:08:57'
published: true
tags:
- Proxmox
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2025-03-11_cover.png"
thumbnail-img: ""
---

Este artículo explica cómo instalar [**Docker Engine**](https://docs.docker.com/get-started/get-docker/){:target="_blank"} en un [**contenedor Linux (LXC)**](https://linuxcontainers.org/lxc/introduction/){:target="_blank"}, basado en [**TurnKey Core 18.1**](https://www.turnkeylinux.org/core){:target="_blank"} (**Debian 12**), desplegado en el servidor [**Proxmox VE**](proxmox-ve-802-en-un-dell-optiplex-7050) que instalé hace unos meses.

{: .box-note}
**Nota**: El soporte oficial de Proxmox recomienda ejecutar Docker en máquinas virtuales, pero un contenedor Linux (LXC) permite hacerlo con menos recursos del hipervisor y con velocidades de arranque mucho más rápidas.

# Actualizar Proxmox

Antes de empezar, nos aseguraremos que Proxmox esté correctamente actualizado. Se puede comprobar la versión actual mediante el comando `pveversion`:

```plaintext
root@pve:~# pveversion 
pve-manager/8.3.2/3e76eec21c4a14a7 (running kernel: 6.8.12-6-pve)
```

A continuación se procede a actualizar el servidor usando los comandos propios de Proxmox:

```plaintext
# Actualizar repositorios
pveupdate

# Actualizar paquetes de Proxmox
pveupgrade
```

{: .box-note}
Si al ejecutar `pveupgrade` aparece un mensaje indicando que "los **paquetes de GRUB** no pueden actualizar el _bootloader_ en `/boot/efi/EFI/BOOT/BOOTX64.efi`" únicamente hay que [cambiar la configuración de `grub-efi-amd64`](actualizacion-de-bootx64efi-en-proxmox) y volver a reinstalarlo.

A continuación se reinicia el servidor para aplicar los cambios (sobretodo si ha habido alguna actualización del _kernel_):

```plaintext
# Reiniciar para aplicar cambios
reboot -h now
```

Si todo funciona correctamente, el servidor debería reiniciar con la nueva versión del _kernel_ y de la interfaz web:

```plaintext
root@pve:~# pveversion 
pve-manager/8.3.4/65224a0f9cd294a3 (running kernel: 6.8.12-8-pve)
```

# Descargar plantilla de TurnKey Core

Antes de poder crear un contenedor Linux (LXC) es necesario descargar la plantilla del SO que ejecutaremos en él. En este caso se utilizará [**TurnKey Core**](https://www.turnkeylinux.org/core){:target="_blank"}, una pequeña distribución basada en **Debian GNU/Linux**.

Se pueden [actualizar y descargar plantillas LXC](como-actualizar-las-plantillas-lxc-de-proxmox) usando el comando `pveam` tal como se muestra a continuación:

```bash
pveam update
pveam available --section turnkeylinux  | grep -i "core"
pveam download local debian-12-turnkey-core_18.1-1_amd64.tar.gz
```

{: .box-note}
TurnKey Core 18.1 está basada en Debian 12 y fue publicada el 29 de julio de 2024 según indican sus [_manifest_ & _signs_](https://releases.turnkeylinux.org/turnkey-core/18.1-bookworm-amd64/){:target="_blank"}.

# Contenedor Linux (CT)

## Instalación

A continuación se puede crear el contenedor Linux (**CT** en terminología Proxmox) usando la interfaz gráfica o, mejor aún, [desde el CLI de Proxmox usando el comando `pct create`](crear-un-lxc-desde-el-cli-de-proxmox).

```plaintext
root@pve:~# ./create_ct.sh 
Introduce el CT ID: 701
Introduce el nombre del contenedor: dockerlxc
Privilegiado (yes/no) [no]: 
¿Utilizarás Docker? (yes/no) [no]: yes
Introduce el password del usuario 'root': 
Confirma el password del usuario 'root': 
Introduce el tamaño del disco (GB) [8]: 10
Introduce la memoria RAM (MB) [512]: 1024
Introduce el número de cores [1]: 1
Introduce la dirección IP (192.168.1.x): 192.168.1.99
Introduce los servidores DNS (separados por espacios) [192.168.1.81 192.168.1.82]: 
¿Quieres montar /datos/samba/media en el CT? (yes/no) [no]: 
¿Quieres montar /backups/rsync en el CT? (yes/no) [no]: 
Resumen de datos:
CT ID: 701
Nombre del contenedor: dockerlxc
Privilegiado: no
Utilizará Docker: yes
Password: [oculto]
Tamaño del disco: 10GB
Memoria RAM: 1024MB
Cores: 1
IP: 192.168.1.99
Servidores DNS: 192.168.1.81 192.168.1.82
Montar /datos/samba/media: no
Montar /backups/rsync: yes
Memoria SWAP: 512MB
Gateway: 192.168.1.1
¿Son correctos estos datos? (yes/no) [no]: yes
```

## Ejecución

El contenedor se configura para que se ponga en marcha automáticamente al iniciarse el servidor Proxmox y se pone en marcha a continuación:

```plaintext
pct set 701 --onboot 1
pct start 701
```

Una vez puesto en marcha, se puede entrar en su _shell_ para ver el proceso de instalacón:

* Expandir el centro de datos (_Datacenter_)
* Expandir el servidor (_node_): `pve`
* Seleccionar el contenedor (_container_): `701 (dockerlxc)`
* Pulsar el botón **Console** para ver la instalación

Una vez iniciado el contenedor Linux, se puede iniciar sesión:

* dockers login: `root`
* Password: `********`

Cuando aparezca el asistente de instalación **TurnKey GNU/Linux - First boot configuration**:

* Pulsar el botón **Skip** en la pantalla _Initialize Hub Services_
* Pulsar el botón **Skip** en la pantalla _System Notifications and Critical Security Alerts_
* Pulsar el botón **Install** en la pantalla _Security updates_

{: .box-note}
Si aparece el mensaje `A security update to the kernel requires a reboot to go into effect` se reiniciará el LXC despues de salir del asistente.

Al finalizar la instalación de las actualizaciones de seguridad, en la _TurnKey GNU/Linux Configuration Console_ se mostrarán las URLs de servicio, por ejemplo:

```plaintext
Webmin: https://192.168.1.99:12321
SSH/SFTP: root@192.168.1.99 (port 22)
```

En esta pantalla hay que:

* Pulsar el botón **Advanced Menu**
* Seleccionar la opción **Quit** para salir de la consola de configuración
* Pulsar el botón **Yes** para confirmar

## Actualización

{: .box-note}
**Nota**: Aunque no es estrictamente necesario, se recomienda cambiar la zona horaria antes de seguir. Para ello se puede utilizar el comando `dpkg-reconfigure tzdata` y seleccionar `Europe/Madrid`.

Se actualizan los repositorios y los paquetes de la distribución Debian usando los siguientes comandos:

```plaintext
apt update
apt upgrade -y
```

Durante la instalación pueden aparecer pantallas relacionadas con `postfix` y `openssh-server`:

* En la pantalla _Postfix Configuration_, seleccionar la opción **No configuration** y pulsar el botón **OK**
* En la pantalla _Configuring openssh-server_, seleccionar la opción **Keep the local version currently installed** para conservar el fichero `sshd_config` y pulsar el botón **OK**

{: .box-note}
**Nota**: En este momento es recomendable reiniciar el LXC usando el comando `reboot` (sobretodo si se han actualizado paquetes críticos o el _kernel_).

# Docker Engine

## Instalación

La [instalación de **Docker Engine**](https://docs.docker.com/engine/install/debian/#install-using-the-repository){:target="_blank"} se realiza desde el repositorio `apt` oficial ejecutando los siguientes comandos:

```bash
# Add Docker's official GPG key:
apt update
apt install ca-certificates curl
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  tee /etc/apt/sources.list.d/docker.list > /dev/null
apt update

# Install latest Docker packages
apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## Comprobar servicio `docker.service`

Para comprobar que Docker se ha instalado correctamente en el LXC, se puede ejecutar el comando `systemctl status docker` que muestra información acerca del servicio:

```plaintext
root@dockerlxc ~# systemctl status docker
* docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; preset: enabled)
     Active: active (running) since Wed 2025-03-12 19:07:06 CET; 26s ago
TriggeredBy: * docker.socket
       Docs: https://docs.docker.com
   Main PID: 40454 (dockerd)
      Tasks: 7
     Memory: 20.6M
        CPU: 208ms
     CGroup: /system.slice/docker.service
             `-40454 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

## Ejecutar aplicación `hello-world`

Para comprobar que todo funciona correctamente, se puede ejecutar el comando `docker run hello-world`:

```plaintext
root@dockerlxc ~# docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
e6590344b1a5: Pull complete 
Digest: sha256:bfbb0cc14f13f9ed1ae86abc2b9f11181dc50d779807ed3a3c5e55a6936dbdd5
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
```

# Seguridad (opcional)

A continuación se detallan algunos pasos que, aunque no son estrictamente necesarios, aumentan la seguridad del entorno:

* Creación de un usuario normal para ejecutar los contenedores Docker
* Creación de un directorio protegido para hacer copias de seguridad de los datos de los contenedores Docker

## Crear un usuario normal

En mi caso, el usuario normal se llamará `manel` y se creará de la siguiente manera:

```bash
useradd -m -u 1000 -U -G docker -s /bin/bash manel
cat /etc/passwd | grep -i manel
cat /etc/group | grep -i manel
passwd manel
```

## Crear un grupo `rsync`

El contenedor LXC tiene montado en el directorio `/mnt/rsync` el directorio `/backups/rsync` de Proxmox que está ubicado en un disco diferente al utilizado para los contenedores y las máquinas virtuales.

La idea es crear un directorio específico para esta máquina en `/mnt/rsync/dockerlxc`, protegido para el grupo `rsync` del cual será miembro el usuario `manel` creado anteriormente.

{: .box-note}
El grupo `rsync` tendrá un GID relacionado con el identificador del CT mediante la siguiente fórmula: `1000 + CT ID` para evitar solapamientos entre diferentes LXC.

```bash
# 1000 + 701 = 1701
groupadd -g 1701 rsync
mkdir /mnt/rsync/dockerlxc
chown root:1701 /mnt/rsync/dockerlxc
chmod 770 /mnt/rsync/dockerlxc
usermod -aG rsync manel
cat /etc/group | grep -i manel
```

## Instalación y configuración de `sudo`

El usuario `manel` podrá ejecutar contenedores Docker por ser miembro del grupo `docker`. Aunque este usuario puede ejecutar el comando `rysnc`, a la hora de copiar ficheros de los contenedores Docker creados por `root` tendrá problemas de acceso.

Para solucionarlo, hay que instalar el paquete `sudo` y configurarlo para el usuario `manel` pueda ejecutar `sudo rsync` y copiar así cualquier ficheros protegido.

* Instalar el paquete: `apt install sudo -y`
* Añadir la siguiente línea al fichero `/etc/sudoers`

```plaintext
manel ALL=(ALL) NOPASSWD: /usr/bin/rsync
```

## Iniciar sesión con el usuario normal

A continuación se inicia sesión con el usuario `manel` para instalar un par de contenedores Docker que pueden ser de utilidad en el futuro: **Portainer** y **Nginx Proxy Manager**.

Se puede iniciar sesión mediante SSH o usar el comando `su - manel` para convertirnos en el usuario normal mediante el cual se ejecutarán los contenedores Docker en esta máquina.

# Instalar **Nginx Proxy Manager** (opcional)

Instalar [Nginx Proxy Manager](https://nginxproxymanager.com/){:target="_blank"} es una opción muy recomendable para exponer los servicios de forma fácil y segura mediante certificados SSL emitidos por Let's Encrypt.

Como ya expliqué la [instalación de Nginx Proxy Manager en Docker](instalacion-de-nginx-proxy-manager-en-docker) de forma detallada, aquí únicamente indicaré los comandos para hacerlo rápidamente:

* Crear el directorio para Nginx Proxy Manager: `mkdir -p ~/dockers/nginx-proxy-manager`
* Crear el directorio para el volumen de datos de NPM: `mkdir -p ~/dockers/nginx-proxy-manager/nginx-proxy-manager_data`
* Crear el directorio para el volumen de datos de Let's Encrypt: `mkdir -p ~/dockers/nginx-proxy-manager/nginx-proxy-manager_letsencrypt`
* Crear el fichero `docker-compose.yml` en el directorio `~/dockers/nginx-proxy-manager` con el siguiente contenido:

```yaml
services:
  nginx-proxy-manager:
    image: jc21/nginx-proxy-manager:2.12.3
    container_name: nginx-proxy-manager
    environment:
      TZ: 'Europe/Madrid'
      DISABLE_IPV6: 'true'
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - ./nginx-proxy-manager_data:/data
      - ./nginx-proxy-manager_letsencrypt:/etc/letsencrypt
    ports:
      - 80:80 # Public HTTP Port
      - 443:443 # Public HTTPS Port
      - 81:81 # Admin Web Port
    restart: unless-stopped

networks:
  default:
    external: true
    name: nginx
```

A continuación, se crea una red específica para ejecutar todos los contenedores Docker de esta máquina:

* `docker network create nginx`

Y, finalmente, se pone en marcha el contenedor Docker:

* `docker compose up -d`

{: .box-note}
Es necesario cambiar el usuario (`admin@example.com`) y la contraseña por defecto (`changeme`) accediendo al puerto 81 del contenedor LXC: <http://192.168.1.99:81/>{:target="_blank"}.

# Instalar **Portainer CE** (opcional)

Instalar [Portainer Community Edition (CE)](https://docs.portainer.io/){:target="_blank"} es una opción muy recomendable para poder administrar los contenedores Docker de una forma más sencilla.

Como ya expliqué la [instalación de Portainer CE en Docker](instalacion-de-portainer-ce-en-docker) de forma detallada, aquí únicamente indicaré los comandos para hacerlo rápidamente:

* Crear el directorio para Portainer: `mkdir -p ~/dockers/portainer-ce`
* Crear el directorio para el volumen de datos: `mkdir -p ~/dockers/portainer-ce/portainer-ce_data`
* Crear el fichero `docker-compose.yml` en el directorio `~/dockers/portainer-ce` con el siguiente contenido:

```yaml
services:
  portainer-ce:
    image: portainer/portainer-ce:2.27.1
    container_name: portainer-ce
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock
      - ./portainer-ce_data:/data
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true

networks:
  default:
    external: true
    name: nginx
```

A continuación, se pone en marcha el contenedor Docker:

* `docker compose up -d`

## Configuración del proxy host

Para poder acceder al contenedor Docker de Portainer CE es necesario añadir un **proxy host** en Nginx Proxy Manager:

* Acceder a **Hosts** &rarr; **Proxy Hosts**
* Pulsar el botón `Add Proxy Host`
* Configurar un nuevo proxy host:
  * Details
    * Domain Names: `portainer.mydomain.com` (será necesario añadirlo a nuestro DNS)
    * Scheme: `https`
    * Forward Hostname: `portainer-ce` (el nombre del Docker)
    * Forward Port: `9443`
    * Block Common Exploits: Habilitado
  * SSL
    * Request a new SSL certificate
    * Force SSL: Habilitado
    * HTTP/2 Support: Habilitado
    * HSTS Enabled: Habilitado
    * USE a DNS Challenge: Habilitado
    * DNS Provider: **Cloudflare**
    * Credentials File Content: `dns_cloudflare_api_token = xxxxxxxxxxxxxxxxxxxx`
    * Email Address for Let's Encrypt: `nginx@mydomain.com`
    * I Agree to the Let's Encrypt Terms of Service
  * Pulsar el botón `Save`

Si todo funciona correctamente, se creará un certificado SSL mediante `certbot` y se asignará al **proxy host** `portainer.mydomain.com`. El comando ejecutado por Nginx Proxy Manager es el siguiente:

```plaintext
certbot certonly \
--config '/etc/letsencrypt.ini' --work-dir "/tmp/letsencrypt-lib" --logs-dir "/tmp/letsencrypt-log" \
--cert-name 'npm-1' --agree-tos --email 'nginx@mydomain.com' \
--domains 'portainer.mydomain.com' \
--authenticator 'dns-cloudflare' --dns-cloudflare-credentials '/etc/letsencrypt/credentials/credentials-1
```

{: .box-note}
Para poder crear el certificado SSL, es necesario que en el panel de configurción DNS de Cloudflare se haya definido una entrada de tipo `A` para `portainer.mydomain.com` que apunte a la dirección IP del contenedor LXC (en este caso, `192.168.1.99`).

Si todo es correcto, se podrá acceder a <https://portainer.mydomain.com/>{:target="_blank"} donde aparecerá la pantalla para crear el usuario administrador inicial y su contraseña antes de poder comenzar a utilizar el entorno.

# Backups mediante `rsync`

Finalmente, se crea el fichero [`~/dockers/backup_dockers.sh`](https://gist.github.com/manelrodero/a693616deacf2a04e93f9959a7a6ee2e){:target="_blank"} y se programa su ejecución mediante **cron** a la hora que se crea conveniente:

```plaintext
# m h  dom mon dow   command
30 2 * * * /home/manel/dockers/backup_dockers.sh
```

### Historial de cambios

* **2025-03-11**: Documento inicial
