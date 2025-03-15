---
layout : post
blog-width: true
title: 'Instalación de Home Assistant en Proxmox'
date: '2023-11-25 16:31:58'
published: true
tags:
- Proxmox
- Docker
- HomeLab
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2023-11-25_cover.png"
thumbnail-img: ""
---

Para seguir el proceso de migración de mi _Home Lab_ desde una pequeña [Raspberry Pi 4](instalar-raspberry-pi-os-64bits) a un [OptiPlex 7050 ejecutando Proxmox](proxmox-ve-802-en-un-dell-optiplex-7050) ahora le toca el turno a [**Home Assistant**](https://www.home-assistant.io/){:target="_blank"}, un sistema de domótica de código abierto que prioriza el control local y la privacidad.

En esta ocasion, la [instalación de Home Assistant en Docker](instalacion-de-home-assistant-en-docker) se realizará usando una máquina virtual Ubuntu en Proxmox. Esta máquina virtual estará basada en la [plantilla de Ubuntu Minimal 22.04 LTS](creacion-de-una-plantilla-de-ubuntu-en-proxmox-ve) que había creado anteriormente.

# Creación de la Máquina Virtual

La creación de la máquina virtual se realiza [clonando la plantilla de Ubuntu](creacion-de-una-plantilla-de-ubuntu-en-proxmox-ve#uso-de-la-plantilla) desde la consola de Proxmox:

```bash
qm clone 701 100 --name smarthome --full 1 --storage local-lvm
```

El comando anterior crea una VM llamada `smarthome` con el identificador `100` que no dependerá de la plantilla por haberse realizado un _full clone_ y el disco de la misma se guardará en `local-lvm`.

A continuación se modifican sus características:

* Hardware
  * Memory: **4096MB**
  * Processors: 1 socket, **2 cores**
* Cloud-Init
  * User: **manel**
  * Password: `********`

{: .box-note}
**Nota**: Como se han cambiado algunos valores de _cloud init_, es necesario regenerar la imagen antes de iniciar la máquina virtual.

A continuación se accede a la consola de la VM y se pone en marcha para proceder a instalar el [agente de QEMU](https://pve.proxmox.com/wiki/Qemu-guest-agent){:target="_blank"}:

```bash
sudo apt install qemu-guest-agent -y
sudo shutdown -h now
```

Una vez apagada la máquina, se procede a configurar la dirección IP estática y el DNS de la máquina:

* Cloud-Init
  * DNS domain: **home**
  * DNS servers: **192.168.1.81 192.168.1.82**
  * IP Config (net0): **Static**
    * IPv4/CIDR: **192.168.1.100/24**
    * Gateway: **192.168.1.1**

{: .box-note}
**Nota**: Estos valores de _cloud init_ no se pueden cambiar inicialmente porque entonces aparece el error `Failed to start execute cloud user/final scripts` y es imposible acceder a la máquina.

# Configuración de la VM

Antes de seguir con la instalación de Docker y Home Assistant, es necesario instalar algunos paquetes para poder configurar correctamente la máquina:

```bash
sudo apt install dialog nano -y

# Seleccionar zona Europe/Madrid
sudo dpkg-reconfigure tzdata
```

# Instalación de Docker Engine

La [instalación de **Docker Engine**](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository){:target="_blank"} se realiza desde el repositorio `apt` oficial ejecutando los siguientes comandos:

```bash
# Add Docker's official GPG key:
sudo apt install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the repository to Apt sources:
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update

# Install latest Docker packages
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

Para que mi usuario que no es `root` pueda ejecutar Docker correctamente hay que añadirlo al grupo correspondiente (hay que tener en cuenta posibles [problemas de seguridad](https://docs.docker.com/engine/security/security/#docker-daemon-attack-surface){:target="_blank"}):

```bash
sudo usermod -aG docker $(whoami)
```

Finalmente, se reinicia:

```bash
sudo reboot
```

Y cuando haya reiniciado, se puede comprobar la versión mediante el comando `docker version`. En el momento de escribir este artículo la última versión disponible es la 24.0.7.

# Instalación de Home Assistant

La [instalación en Docker](https://hub.docker.com/r/linuxserver/homeassistant){:target="_blank"} se realiza usando la imagen `linuxserver/homeassistant`.

La forma más sencilla es usar un fichero `docker-compose.yml` con el siguiente contenido:

```yaml
version: '3'
services:
  homeassistant:
    image: ghcr.io/home-assistant/home-assistant:stable
    container_name: homeassistant
    environment:
      - TZ=Europe/Madrid
    volumes:
      - ./homeassistant_config:/config
      - /etc/localtime:/etc/localtime:ro
    restart: always
    privileged: true
    network_mode: host
```

## Configuración de HA

na vez en marcha, se puede acceder a Home Assistant a través del puerto `8123` (en este ejemplo [http://192.168.1.100:8123](http://192.168.1.100:8123)) y comenzar la configuración pulsando el botón **Create my Smart Home**.

{: .box-note}
**Nota**: Se configura de la misma manera que cuando realicé la [instalación en la Raspberry Pi 4](instalacion-de-home-assistant-en-docker).

# ¿Y ahora qué?

Los siguientes pasos serían similares a los realizados en la Raspberry Pi 4:

* Instalar [Network UPS Tools](https://networkupstools.org/){:target="_blank"}
* Integrar `NUT` en Home Assistant
* Instalar [HACS](https://hacs.xyz/){:target="_blank"}
* Instalar [Mosquitto](https://www.mosquitto.org/){:target="_blank"}
* Instalar [Zigbee2MQTT](https://www.zigbee2mqtt.io/){:target="_blank"}
* Etc.

# HACS

La instalación de HACS es muy sencilla y se puede realizar desde el propio Docker de HA:

```bash
# Acceder al Docker
docker exec -it homeassistant /bin/bash

# Dentro del Docker
cd /config
wget -O - https://get.hacs.xyz | bash -
exit
```

A continuación se reinicia el contenedor y se vuelve a iniciar sesión para finalizar la [configuración de HACS tal como se hizo en la Raspberry Pi 4](instalacion-de-hacs-en-home-assistant-docker).

# Referencias

* [Why does /etc/resolv.conf point at 127.0.0.53?](https://unix.stackexchange.com/questions/612416/why-does-etc-resolv-conf-point-at-127-0-0-53){:target="_blank"}
* [Monitorizar un SAI (UPS) desde una Raspberry Pi](monitorizar-un-sai-ups-desde-una-raspberry-pi)
* [Integrar NUT en Home Assistant](integrar-nut-en-home-assistant)
* [Instalación de HACS en Home Assistant (Docker)](instalacion-de-hacs-en-home-assistant-docker)
* [Instalación de Mosquitto (MQTT Broker) en Docker](instalacion-de-mosquitto-mqtt-broker-en-docker)
* [Instalación de Zigbee2MQTT en Docker](instalacion-de-zigbee2mqtt-en-docker)
* [Migración de Home Assistant Container a Home Assistant OS](https://www.youtube.com/watch?v=COlxhE6639c){:target="_blank"}
