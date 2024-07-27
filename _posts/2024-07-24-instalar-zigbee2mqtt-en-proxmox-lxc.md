---
layout : post
blog-width: true
title: 'Instalar Zigbee2MQTT en Proxmox LXC'
date: '2024-07-24 22:31:46'
#last-updated: '2024-07-24 22:31:46'
published: true
tags:
- Proxmox
- Home Assistant
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2024-07-24_cover.png"
thumbnail-img: ""
---

Para seguir el proceso de migración de mi _Home Lab_ desde una pequeña [Raspberry Pi 4](instalar-raspberry-pi-os-64bits) a un [OptiPlex 7050 ejecutando Proxmox](proxmox-ve-802-en-un-dell-optiplex-7050) ahora le toca el turno a [**Zigbee2MQTT**](https://www.zigbee2mqtt.io/){:target="_blank"}, un _bridge_ entre los dispositivos Zigbee y MQTT que no necesita acceso a un _cloud_ propietario para funcionar.

En esta ocasion, en lugar de realizar la [instalación de Zigbee2MQTT en Docker](instalacion-de-zigbee2mqtt-en-docker), se utilizará un contenedor ligero LXC de Proxmox con Debian 12.

# Creación del LXC

Este _bridge_ **Zigbee** con [Zigbee2MQTT](https://www.zigbee2mqtt.io/){:target="_blank"} se ejecutará sobre un LXC creado desde la _Shell_ del servidor Proxmox mediante los [Proxmox VE Helper-Scripts](https://tteck.github.io/Proxmox/){:target="_blank"}:

```
mkdir -p ~/tteck && cd ~/tteck
wget -nv https://github.com/tteck/Proxmox/raw/main/ct/zigbee2mqtt.sh
# Revisar el script ;-)
bash mqtt.sh
```

Al ejecutar el _script_ anterior, aparecerá el asistente de instalación:

* This will create a New Zigbee2MQTT LXC. Proceed? `Yes`
* Use Default Settings? `Advanced`
* To make a selection, use the Spacebar. `Ok`
* If the default Linux distribution is not adhered to, script support will be discontinued (debian 12). `Ok`
* Choose Distribution: **debian**
* Choose version: **12 Bookworm**
* Choose Type: **1 Unprivileged**
* Set Root Password (needed for root ssh access): **xxxxxxxx**
* Verify Root Password: **xxxxxxxx**
* Set Container ID: **309**
* Set Hostname: **zigbee2mqtt**
* Set Disk Size in GB: **4**
* Allocate CPU Cores: **2**
* Allocate RAM in MiB: **1024**
* Set a Bridge: **vmbr0**
* Set a Static IPv4 CIDR Address (/24): **192.168.1.86/24**
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
* Ready to create Zigbee2MQTT LXC? `Yes`

{: .box-note}
Deshabilitar IPv6 implica añadir la siguiente configuración al LXC: `net.ipv6.conf.all.disable_ipv6 = 1`

Durante la clonación del repositorio de Zigbee2MQTT, el _script_ preguntará si se quiere cambiar a la rama Edge/dev y la respuesta será `N` para utilizar la **rama estable**:

```
 ✓ Set up Zigbee2MQTT Repository
Switch to Edge/dev branch? (y/N) N
npm warn deprecated inflight@1.0.6: This module is not supported, and leaks memory. Do not use it.
Check out lru-cache if you want a good and tested way to coalesce async requests by a key value, which is much more comprehensive and powerful.
npm warn deprecated glob@7.2.3: Glob versions prior to v9 are no longer supported
npm warn deprecated @humanwhocodes/object-schema@2.0.3: Use @eslint/object-schema instead
npm warn deprecated @humanwhocodes/config-array@0.11.14: Use @eslint/config-array instead
npm warn deprecated rimraf@3.0.2: Rimraf versions prior to v4 are no longer supported

added 798 packages, and audited 799 packages in 13s

87 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
npm notice
npm notice New patch version of npm available! 10.8.1 -> 10.8.2
npm notice Changelog: https://github.com/npm/cli/releases/tag/v10.8.2
npm notice To update run: npm install -g npm@10.8.2
npm notice
 ✓ Installed Zigbee2MQTT
 /Created symlink /etc/systemd/system/multi-user.target.wants/zigbee2mqtt.service → /etc/systemd/system/zigbee2mqtt.service.
 ✓ Created Service
```

{: .box-note}
Los mensajes de error de NPM son debidos a **dependencias indirectas** según se explica en la [_issue_ 23238](https://github.com/Koenkk/zigbee2mqtt/issues/23238#issuecomment-2204103062){:target="_blank"} del proyecto.

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
Al deshabilitar _nesting_ en un [LXC Debian 12](https://pve.proxmox.com/pve-docs/pct.conf.5.html){:target="_blank"}, el acceso a la _Console_ del LXC es unos segunods más lento.

En este punto se puede conectar, en un puerto USB del servidor Proxmox, un [adaptador Zigbee](https://www.zigbee2mqtt.io/guide/adapters/){:target="_blank"} que hará las funciones de coordinador de la red Zigbee.

En mi caso usaré un [SONOFF ZigBee 3.0 USB Dongle Plus](https://sonoff.tech/product/gateway-amd-sensors/sonoff-zigbee-3-0-usb-dongle-plus-p/){:target="_blank"} que se puede comprar fácilmente en [AliExpress](https://es.aliexpress.com/item/1005003758328408.html){:target="_blank"}.

Para identificar el dispositivo que se ha conectado se puede utilizar el comando `ls -l /dev/serial/by-id` tal como se muestra a continuación:

```
root@pve:~# ls -l /dev/serial/by-id/
lrwxrwxrwx 1 root root 13 Jul 24 23:18 usb-Silicon_Labs_Sonoff_Zigbee_3.0_USB_Dongle_Plus_0001-if00-port0 -> ../../ttyUSB0

root@pve:~# ls -l /dev/ttyUSB0 
crw-rw---- 1 root dialout 188, 0 Jul 24 23:18 /dev/ttyUSB0
```

Para realizar el _passthrough_ del dispositivo USB al contenedor LXC hay que editar su fichero de configuración `/etc/pve/lxc/309.conf` y añadir las siguientes líneas:

```
lxc.cgroup2.devices.allow: c 188:* rwm
lxc.mount.entry: /dev/serial/by-id  dev/serial/by-id  none bind,optional,create=dir
lxc.mount.entry: /dev/ttyUSB0       dev/ttyUSB0       none bind,optional,create=file
```

A continuación, se reinicia el LXC mediante el comando `pct reboot 309` desde la _Shell_ de Proxmox o usando el botón `Reboot` desde la interfaz gráfica del propio LXC.

# Configuración del LXC

Al no haber habilitado el acceso para el usuario `root` mediante SSH, se puede acceder al LXC utilizando el botón `Console` desde la configuración del propio LXC.

Una vez dentro del LXC, se procederá a realizar una configuración similar a la realizada durante la [instalación de Zigbee2MQTT en Docker](instalacion-de-zigbee2mqtt-en-docker).

El primer paso será modificar el fichero de configuración `/opt/zigbee2mqtt/data/configuration.yaml` para añadir los mismos parámetros del entorno de producción actual en Docker.

Para ello, se hace una copia de seguridad del fichero actual, se copia el fichero de configuración desde la Raspberry y se modifica el fichero usando el editor `nano`:

```
cd /opt/zigbee2mqtt/data
mv configuration.yaml configuration.original
scp pi@pi:/home/pi/volumes/zigbee2mqtt/configuration.yaml configuration.pi
nano configuration.yaml
```

El contenido final del fichero de configuración `configuration.yaml` será parecido al siguiente:

```
homeassistant: true
permit_join: false
mqtt:
  base_topic: zigbee2mqtt
  server: mqtt://192.168.1.85
  user: admin
  password: xxxxxxxx
serial:
  port: /dev/serial/by-id/usb-Silicon_Labs_Sonoff_Zigbee_3.0_USB_Dongle_Plus_0001-if00-port0
frontend:
  auth_token: xxxxxxxx
advanced:
  homeassistant_legacy_entity_attributes: false
  legacy_api: false
  legacy_availability_payload: false
  network_key:
    - xxx
    - xxx
    - xxx
    - xxx
    - xxx
    - xxx
    - xxx
    - xxx
    - xxx
    - xxx
    - xxx
    - xxx
    - xxx
    - xxx
    - xxx
    - xxx
device_options:
  legacy: false
devices:
  '0x0000000000000000':
    friendly_name: sensor_temperatura
    temperature_precision: 1
    humidity_precision: 1
  '0x0000000000000000':
    friendly_name: sensor_apertura

[...]

ota:
  ikea_ota_use_test_url: false
  disable_automatic_update_check: true    
```

{: .box-note}
En el fichero anterior se han ocultado los valores sensibles como la **contraseña de MQTT**, la **contraseña del _frontend_ de Zigbee2MQTT**, la **clave de la red Zigbee** o los **dispositivos Zigbee** existentes en la misma.


A continuación se copian algunos ficheros que contienen el estado actual de la red Zigbee:

```
scp pi@pi:/home/pi/volumes/zigbee2mqtt/database.db .
scp pi@pi:/home/pi/volumes/zigbee2mqtt/coordinator_backup.json .
scp pi@pi:/home/pi/volumes/zigbee2mqtt/state.json .
```

Finalmente, se puede poner en marcha **Zigbee2MQTT** usando el comando `npm start` desde el directorio del proyecto

```
cd /opt/zigbee2mqtt && npm start
```

Si aparece algún error indicando que no se puede acceder al dispositivo `/dev/ttyUSB0` es debido a los **permisos de usuario/grupo** en Proxmox y en LXC.

Para que el usuario `root` del LXC pueda acceder a `/dev/ttyUSB0` es necesario cambiar el propietario en Proxmox usando el siguiente comando:

```
# Desde Proxmox
chown 100000:100000 /dev/ttyUSB0
```

Antes de ejecutar este comando, los permisos eran los siguientes:

```
# En Proxmox
root@pve:~# ls -l /dev/ttyUSB0 
crw-rw---- 1 root dialout 188, 0 Jul 24 23:18 /dev/ttyUSB0

# En el LXC
root@zigbee2mqtt:~# ls -l /dev/ttyUSB0 
crw-rw---- 1 nobody nogroup 188, 0 Jul 24 23:18 /dev/ttyUSB0
```

Después de ejecutar el comando `chown`, los permisos quedan de la siguiente manera:

```
# En Proxmox
root@pve:~# ls -l /dev/ttyUSB0 
crw-rw---- 1 100000 100000 188, 0 Jul 24 23:18 /dev/ttyUSB0

# En el LXC
root@zigbee2mqtt:~# ls -l /dev/ttyUSB0 
crw-rw---- 1 root root 188, 0 Jul 24 23:18 /dev/ttyUSB0
```

# Frontend

El acceso al _frontend_ de Zigbee2MQTT mediante la URL http://192.168.1.86:8080 debería funcionar correctamente usando la misma contraseña del entorno anterior.

# Integración con Home Assistant

La integración con Home Assistant a través de **MQTT Discovery** también debería funcionar correctamente gracias a la configuración copiada desde el entorno anterior.


# Actualizar

La actualización de Zigbee2MQTT en LXC es algo más complicada que en Docker (donde únicamente hay que parar el Docker, borrarlo y volver a ejecutarlo mediante `docker compose`):


```
# Parar Zigbee2MQTT y acceder a su directorio
systemctl stop zigbee2mqtt
cd /opt/zigbee2mqtt

# Backup configuration
cp -R data /root/zigbee2mqtt_backup

# Update
git pull
npm ci

# Restore configuration
cp -R /root/zigbee2mqtt_backup/* data
rm -rf /root/zigbee2mqtt_backup

# Start Zigbee2MQTT
systemctl start zigbee2mqtt
```

# Referencias

* [How to Separate Zigbee2MQTT from Home Assistant in Proxmox](https://smarthomescene.com/guides/how-to-separate-zigbee2mqtt-from-home-assistant-in-proxmox/){:target=_blank}

### Historial de cambios

* **2024-07-24**: Documento inicial
* **2024-07-27**: Se añade fichero de configuración
