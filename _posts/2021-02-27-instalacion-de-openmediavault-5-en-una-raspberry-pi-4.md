---
layout : post
blog-width: true
title: 'Instalación de OpenMediaVault 5 en una Raspberry Pi 4'
date: '2021-02-21 16:16:00'
published: true
tags:
- Raspberry
author:
  display_name: Manel Rodero
---

La instalación de [OpenMediaVault 5](https://www.openmediavault.org/){:target="_blank"} en una Raspberry Pi 4 está basada en la [instalación para Debian](https://openmediavault.readthedocs.io/en/5.x/installation/on_debian.html){:target="_blank"} pero hay que utilizar un [script](https://github.com/OpenMediaVault-Plugin-Developers/installScript){:target="_blank"} para que se realice correctamente.

```bash
# Borrar el enlace a /dev/null
sudo rm -f /etc/systemd/network/99-default.link

# Reiniciar
sudo reboot

# Descargar script de instalación
curl -fsSL https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install -o install-omv5.sh

# Instalar sin el flashmemory plugin
chmod u+x install-omv5.sh
sudo ./install-omv5.sh -f
```

Este script puede tardar 10-15 minutos en ejecutarse dependiendo de factores como la conexión a Internet, la velocidad de escritura del dispositivo USB utilizado, etc.

{: .box-warning}
**Atención**: No cerrar la conexión SSH con PuTTY. Cuando el script finalice, la Raspberry Pi se reiniciará automáticamente después de mostrar el siguiente mensaje:

```plaitext
IP address may change and you could lose connection if running this script via ssh.
[...]
Network setup for DHCP.  Rebooting...
```

Cuando la Raspberry Pi reinicie, se podrá acceder a la interfaz web de OMV con el usuario y contraseña por defecto (U: `admin`, P: `openmediavault`).

Es muy aconsejable **cambiar esta contraseña** accediendo a `System` > `General Settings` > `Web Administrator Password`.

Otras opciones de configuración que se recomienda revisar son las siguientes:

* System
  * General Settings
    * Web Administration
      * Auto logout: 5 minutes &rarr; Disabled
  * Date & Time
    * Time zone: Europe/Madrid
    * Use NTP server: Enabled
    * Time servers: pool.ntp.org
  * Network:
    * Hostname: pi4nas
    * Domain name: home
    * Interfaces eth0:
      * IPv4: DHCP &rarr; Static
      * IPv6: Disabled
      * DNS: 9.9.9.9

## Crear _shares_

La instalación de Raspbian OS tiene, por defecto, dos particiones en el primer disco:

* `/dev/sda1`, boot, vfat
* `/dev/sda2`, rootfs, ext4

Si se añade segundo disco, éste se puede utilizar para almacenar datos:

* Storage
  * File Systems &rarr;  `/dev/sdb1`, data, ext4
* Access Rights Management
  * Shared Folders: PiShare, data , Everyone r/w
* Services
  * SMB/CIFS &rarr; Enable
  * Workgroup: WGLyM
  * Shares: PiShare, Only guests (no username/password)

## Acceder a las carpetas

Para poder acceder a esta carpeta compartida desde un Windows 10 usando la opción _browse network_ es necesario activar la opción **Turn on network discovery** desde `Control Panel\Network and Internet\Network and Sharing Center\Advanced sharing settings`.

Si no se activa esta opción, es necesario mapear la unidad conociendo el nombre `\\pi4nas\PiShare` o la dirección IP `\\192.168.1.180\PiShare`.

{: .box-warning}
**Atención**: Si se produce un error **0x80070035** (_Network path not found_) significa que las políticas de seguridad aplicadas en Windows 10 bloquean el acceso de tipo _unaunthenticated guest_.

Se podría cambiar las políticas del sistema o usar la siguiente clave del registro para "solucionarlo":

```plaintext
[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters]
"AllowInsecureGuestAuth"=dword:1
```

Pero es **más seguro** crear un usuario, asignar ACLs al _Shared Folder_ para este usuario y compartir mediante SBM/CIFS sin permitir usuarios _guests_:

* Access Rights Management
  * User: manel
  * Shared Folders: PiShare &rarr; ACL: manel (read/write)
* Services
  * Shares: PiShare, Public: no

Se pueden comprobar los [ACLs](https://help.ubuntu.com/community/FilePermissionsACLs){:target="_blank"} usando el comando `getfacl`:

```plaintext
pi@pi4nas:~ $ getfacl /srv/dev-disk-by-uuid-c933b03f-a36c-43a4-b02a-80c7119dbcf5/PiShare/
getfacl: Removing leading '/' from absolute path names
# file: srv/dev-disk-by-uuid-c933b03f-a36c-43a4-b02a-80c7119dbcf5/PiShare/
# owner: root
# group: users
# flags: -s-
user::rwx
user:manel:rwx
group::rwx
mask::rwx
other::rwx
default:user::rwx
default:user:manel:rwx
default:group::rwx
default:mask::rwx
default:other::rwx
```

Y después se podría mapear una unidad desde el Explorador de Archivos (`Computer > Map Network Drive`) o desde la línea de comandos:

```plaintext
net use z: \\192.168.1.180\PiShare /user:manel
```

{: .box-note}
**Nota**: En muchas ocasiones el sistema [no escribe correctamente la configuración](https://twitter.com/manelrodero/status/1365715618511679497) y el sistema operativo se queda sin red. Por este motivo se ha decido utilizar [**Samba**](https://www.raspberrypi.org/documentation/remote-access/samba.md) directamente.

## Referencias

* [Raspberry Pi OMV 5 NAS](https://youtu.be/LOg4xfDQafc){:target="_blank"} [ExplainingComputers]
* [Ultimate Media Server : Episode 1 - OpenMediaVault, Docker & Portainer](https://youtu.be/ZLa5NGPKQv0){:target="_blank"} [DB Tech]
* [255: Node-Red, InfluxDB, and Grafana Tutorial on a Raspberry Pi](https://youtu.be/JdV4x925au0){:target="_blank"} [Andreas Spiess]
* [295: Raspberry Pi Server based on Docker](https://www.youtube.com/watch?v=a6mjt8tWUws){:target="_blank"} [Andreas Spiess]
  * : IOTstack: VPN, Dropbox backup, Influx, Grafana, etc.
* [352: Raspberry Pi4 Home Automation Server (incl. Docker, OpenHAB, HASSIO, NextCloud)](https://youtu.be/KJRMjUzlHI8){:target="_blank"} [Andreas Spiess]

## Hardware

* [Orico 2.5" 2-Bay Enclosure UASP](https://www.aliexpress.com/item/1005001816729990.html){:target="_blank"}
* [Orico 2.5" 5-Bay Dock Station](https://www.aliexpress.com/item/1005001723778051.html){:target="_blank"}
