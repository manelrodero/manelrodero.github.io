---
layout : post
blog-width: true
title: 'Instalar Mosquitto (MQTT Broker) en Proxmox LXC'
date: '2024-07-23 20:35:52'
last-updated: '2024-07-23 20:35:52'
published: true
tags:
- Proxmox
- Home Assistant
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2024-07-23_cover.png"
thumbnail-img: ""
---

Para seguir el proceso de migración de mi _Home Lab_ desde una pequeña [Raspberry Pi 4](instalar-raspberry-pi-os-64bits) a un [OptiPlex 7050 ejecutando Proxmox](proxmox-ve-802-en-un-dell-optiplex-7050) ahora le toca el turno a [**Eclipse Mosquitto**](https://mosquitto.org/){:target="_blank"}, un _broker_ de mensajes de código abierto que implementa el protocolo MQTT (_Message Queuing Telemetry Transport_).

En esta ocasion, en lugar de realizar la [instalación de Mosquitto MQTT Broker en Docker](instalacion-de-mosquitto-mqtt-broker-en-docker), se utilizará un contenedor ligero LXC de Proxmox con Debian 12.

# Creación del LXC

Este _broker_ **MQTT** con [Mosquitto](https://mosquitto.org/){:target="_blank"} se ejecutará sobre un LXC creado desde la _Shell_ del servidor Proxmox mediante los [Proxmox VE Helper-Scripts](https://tteck.github.io/Proxmox/){:target="_blank"}:

```
mkdir -p ~/tteck && cd ~/tteck
wget -nv https://github.com/tteck/Proxmox/raw/main/ct/mqtt.sh
# Revisar el script ;-)
bash mqtt.sh
```

Al ejecutar el _script_ anterior, aparecerá el asistente de instalación:

* This will create a New MQTT LXC. Proceed? `Yes`
* Use Default Settings? `Advanced`
* To make a selection, use the Spacebar. `Ok`
* If the default Linux distribution is not adhered to, script support will be discontinued (debian 12). `Ok`
* Choose Distribution: **debian**
* Choose version: **12 Bookworm**
* Choose Type: **1 Unprivileged**
* Set Root Password (needed for root ssh access): **xxxxxxxx**
* Verify Root Password: **xxxxxxxx**
* Set Container ID: **308**
* Set Hostname: **mqtt**
* Set Disk Size in GB: **2**
* Allocate CPU Cores: **1**
* Allocate RAM in MiB: **512**
* Set a Bridge: **vmbr0**
* Set a Static IPv4 CIDR Address (/24): **192.168.1.85/24**
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
* Ready to create MQTT LXC? `Yes`

{: .box-note}
Deshabilitar IPv6 implica añadir la siguiente configuración al LXC: `net.ipv6.conf.all.disable_ipv6 = 1`

Antes de poner en marcha el LXC, se pueden deshabilitar un par de opciones que no son necesarias para el funcionamiento de MQTT y que han sido añadidas por el _script_ ejecutado anteriormente:

* Expandir el centro de datos (_Datacenter_)
* Expandir el servidor (_node_): `pve`
* Seleccionar el contenedor (_container_): `308 (mqtt)`
* Seleccionar la opción **Options**
* Editar los siguientes valores:
  * **Features**:
    * Desmarcar la opción **keyctl**
    * Desmarcar la opción **Nesting**

{: .box-note}
Al deshabilitar _nesting_ en un [LXC Debian 12](https://pve.proxmox.com/pve-docs/pct.conf.5.html){:target="_blank"}, el acceso a la _Console_ del LXC es unos segundos más lento.

A continuación, se reinicia el LXC mediante el comando `pct reboot 308` desde la _Shell_ de Proxmox o usando el botón `Reboot` desde la configuración del propio LXC.

# Configuración del LXC

Al no haber habilitado el acceso para el usuario `root` mediante SSH, se puede acceder al LXC utilizando el botón `Console` desde la interfaz gráfica del propio LXC.

Una vez dentro del LXC, se procederá a realizar una configuración similar a la realizada durante la instalación de [Mosquitto (MQTT Broker) en Docker](instalacion-de-mosquitto-mqtt-broker-en-docker).

El primer paso es crear una contraseña para el usuario `admin` y grabarla en un fichero que se protegerá adecuadamente:

```
# Generar un fichero de password para el usuario admin
mosquitto_passwd -c /etc/mosquitto/passwd admin

# Proteger este fichero para el usuario/grupo de Mosquitto
chown mosquitto:mosquitto /etc/mosquitto/passwd
chmod 440 /etc/mosquitto/passwd
```

A continuación se comprueba que el fichero de configuración `/etc/mosquitto/conf.d/default.conf` contenga la siguiente configuración para deshabilitar el acceso anónimo y establecer el puerto de escucha IPv4:

```
allow_anonymous false
password_file /etc/mosquitto/passwd
listener 1883
socket_domain ipv4
```

Finalmente, se reinicia Mosquitto:

```
systemctl restart mosquitto
```

Y se comprueba que éste inicia correctamente:

```
root@mqtt:/etc/mosquitto/conf.d# tail -f /var/log/mosquitto/mosquitto.log
1721851193: mosquitto version 2.0.11 starting
1721851193: Config loaded from /etc/mosquitto/mosquitto.conf.
1721851193: Opening ipv4 listen socket on port 1883.
1721851193: mosquitto version 2.0.11 running

```

Si es la primera vez que se utiliza MQTT, es recomendable efectuar alguna prueba de publicación de **paquetes** (_packets_) y escucha de **temas** (_topics_) tal como se hizo durante la [instalación de Mosquitto MQTT Broker en Docker](instalacion-de-mosquitto-mqtt-broker-en-docker).

Estas pruebas se pueden realizar fácilmente desde la **integración MQTT** de Home Assistant que se instala de la siguiente manera:

* Acceder a `Settings`
* Acceder a `Device and Services`
* Pulsar en el botón `ADD INTEGRATION`
* Buscar la integración **MQTT** y añadirla
* En el _wizard_ de configuración se introduce la siguiente información:
  * Broker: `192.168.1.85`
  * Port: `1883`
  * Username: `admin`
  * Password: `********`
* Pulsar en `SUBMIT`

Una vez configurada, se puede realizar la prueba de suscripción a un _topic_ y envío de _packets_ desde la configuración de la integración:

* Configure MQTT
* Listen to a topic
  * Topic to subscribe to: `/test/message`
* Pulsar en `START LISTENING`
* Publish a packet
  * Topic: `/test/message`
  * Payload: `Hello world!`
* Pulsar en `PUBLISH`

Si todo funciona correctamente, se debería recibir el mensaje enviado:

```
Message 0 received on /test/message at 9:36 PM:
Hello World!
```

# Referencias

* [How to Separate Zigbee2MQTT from Home Assistant in Proxmox](https://smarthomescene.com/guides/how-to-separate-zigbee2mqtt-from-home-assistant-in-proxmox/){:target="_blank"}
* [Quick Guide to The Mosquitto.conf File With Examples](http://www.steves-internet-guide.com/mossquitto-conf-file/){:target="_blank"}
* [Mosquitto MQTT Cheat Sheet](https://mpolinowski.github.io/docs/Development/Javascript/2021-06-02--mqtt-cheat-sheet/2021-06-02/){:target="_blank"}

### Historial de cambios

* **2024-07-23**: Documento inicial
* **2024-07-27**: Revisión ortográfica
