---
layout : post
blog-width: true
title: 'Sincronizar instancias de Pi-hole usando Gravity Sync'
date: '2024-04-06 12:15:48'
#last-updated: '2024-04-06 12:15:48'
published: true
tags:
- HomeLab
- Proxmox
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2024-04-06_cover.png"
thumbnail-img: ""
---

Este artículo detalla la [instalación y configuración de Gravity Sync](https://github.com/vmstan/gravity-sync/){:target="_blank"} para [sincronizar la configuración de varias instancias de Pi-Hole](https://pi-hole.net/){:target="_blank"}.

# Introducción

Hace unos meses [instalé un par de instancias de Pi-Hole en Proxmox LXC](instalar-pihole-en-proxmox-lxc) para usarlas como servidor DNS y bloquear contenido no deseado (p.ej. publicidad).

Hasta ahora no había tenido la necesidad de sincronizar la configuración de estas instancias pero, al añadir entradas de servicios locales (Proxmox, Unraid, WireGuard, etc.) al DNS de Pi-hole, se hace necesario para no tener que hacerlo en cada una de ellas.

De momento, Pi-hole no incluye ninguna opción para realizar esta sincronización de forma nativa y hay que usar [Gravity Sync](https://github.com/vmstan/gravity-sync/), un proyecto de código abierto, para conseguirlo.

La [versión 4.x de Gravity Sync](https://github.com/vmstan/gravity-sync/wiki/4.0){:target="_blank"} tiene un funcionamiento radicalmente diferente a las versiones previas. El cambio más importante es que todas las instancias se pueden sincronizar entre ellas mediante **_peering_** en lugar de utilizar un modelo de servidor primario y servidor secundario.

{: .box-note}
El _peering_ es interesante en aquellos casos en que los cambios de configuración se pueden realizar en cualquier instancia de Pi-hole.

# Instalación

En mi caso, no se usará la característica de _peering_ y se configurará una de las instancias (servidor `pihole1`) para que realice un **_push_** de la configuración a la otra (servidor `pihole2`).

## Backup de la configuración de Pi-hole

Antes de comenzar, y aunque la instalación no es peligrosa, siempre es interesante realizar una copia de seguridad de los servidores Pi-hole.

Para ello, se ejecuta el comando necesario para realizar una copia de seguridad mediante **Teleporter** en cada uno de los servidores (`pihole1` y `pihole2`):

```Bash
/usr/local/bin/pihole -a -t
```

Este comando genera un fichero `tar.gz` con la fecha y hora de la copia de seguridad en el directorio actual:

```
root@pihole1 ~# ls -la *pi*
-rw-r--r-- 1 root root 2536 Apr  5 21:02 pi-hole-pihole1-teleporter_2024-04-05_21-02-39.tar.gz
```

{: .box-note}
También es posible realizar esta operación desde la interfaz gráfica de Pi-hole (`Settings` &rarr; `Teleporter`). En este caso, el fichero generado se descargará al equipo donde se ejecuta el navegador.

## Descarga del script de instalación

A continuación se descarga el _script_ de instalación `gs-install.sh` en cada uno de los servidores Pi-hole usando el siguiente comando:

```Bash
curl -O -sSL https://raw.githubusercontent.com/vmstan/gs-install/main/gs-install.sh
```

Una vez revisado el contenido del _script_ para comprobar qué hace exactamente (guiño, guiño) se puede proceder a su ejecución en cada uno de los nodos.

## Instalación en `pihole1`

La instalación de Gravity Sync se realiza mediante el siguiente comando:

```Bash
bash gs-install.sh
```

Como este servidor será el **autoritativo** para cualquier cambio en la configuración de las diferentes instancias de Pi-hole de la red, se indicará la IP, usuario y contraseña del servidor `pihole2` en el _wizard_ al final de la instalación tal como se muestra en el siguiente ejemplo:

```
root@pihole1 ~# bash gs-install.sh
∞ Gravity Sync Installation Script
» Validating User Permissions
✓ root is root
✓ Sudo utility detected
» Validating Install of Required Components
✓ SSH has been detected
✓ GIT has been detected
✓ RSYNC has been detected
✓ Systemctl has been detected
» Performing Warp Core Diagnostics
✓ Local installation of Pi-hole has been detected
» Executing Gravity Sync Deployment
∞ Cleaning up bash.bashrc
  You may need to exit your terminal or reboot before running 'gravity-sync' commands
∞ Creating Gravity Sync Directories
Cloning into '/etc/gravity-sync/.gs'...
remote: Enumerating objects: 2808, done.
remote: Counting objects: 100% (248/248), done.
remote: Compressing objects: 100% (148/148), done.
remote: Total 2808 (delta 178), reused 115 (delta 92), pack-reused 2560
Receiving objects: 100% (2808/2808), 621.39 KiB | 3.88 MiB/s, done.
Resolving deltas: 100% (1787/1787), done.
∞ Starting Gravity Sync Configuration
∞ Initializing Gravity Sync (4.0.7)
✓ Evaluating arguments: CONFIGURE
✓ Creating new gravity-sync.conf

  Welcome to the Gravity Sync Configuration Wizard
  Please read through https://github.com/vmstan/gravity-sync/wiki before you continue
  Make sure that Pi-hole is running on this system before your configure Gravity Sync

» Gravity Sync Remote Host Settings
› Remote Pi-hole host address
? IP: 192.168.1.82
✓ Saving 192.168.1.82 host to gravity-sync.conf
› Remote Pi-hole host username
? User: root
✓ Saving root@192.168.1.82 to gravity-sync.conf
» Gravity Sync SSH Key Settings
✓ Generating new SSH key
✓ Moving private key to /etc/gravity-sync/gravity-sync.rsa
✓ Moving public key to /etc/gravity-sync/gravity-sync.rsa.pub
✓ Loading gravity-sync.conf
› Registering SSH key to 192.168.1.82
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/etc/gravity-sync/gravity-sync.rsa.pub"
The authenticity of host '192.168.1.82 (192.168.1.82)' can't be established.
ECDSA key fingerprint is SHA256:6Ry6***********************************027Q.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes

root@192.168.1.82's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh -p '22' 'root@192.168.1.82'"
and check to make sure that only the key(s) you wanted were added.

✓ SSH key registered to 192.168.1.82
» Pi-hole Installation Settings
✓ Detecting local Pi-hole installation
✓ Default install of Pi-hole detected
∞ Detecting remote Pi-hole installation
✓ Detecting remote Pi-hole installation
✓ Remote install of Pi-hole detected

  Configuration has been completed successfully, once Gravity Sync has been installed your other
  node, your next step is to push all of the of data from the currently authoritative
  Pi-hole instance to the other.
  ex: gravity-sync push

  If that completes successfully you can automate future sync jobs to run at a regular interval on
  both of your Gravity Sync peers.
  ex: gravity-sync auto

∞ Gravity Sync CONFIGURE completed after 25 seconds
```

## Instalación en `pihole2`

La instalación de Gravity Sync se realiza mediante el mismo comando utilizado anteriormente:

```Bash
bash gs-install.sh
```

Como este servidor no será el **autoritativo**, se cancelará el _wizard_ al final de la instalación usando `CTRL-C` tal como se muestra en el siguiente ejemplo:


```
root@pihole2 ~# bash gs-install.sh
∞ Gravity Sync Installation Script
» Validating User Permissions
✓ root is root
✓ Sudo utility detected
» Validating Install of Required Components
✓ SSH has been detected
✓ GIT has been detected
✓ RSYNC has been detected
✓ Systemctl has been detected
» Performing Warp Core Diagnostics
✓ Local installation of Pi-hole has been detected
» Executing Gravity Sync Deployment
∞ Cleaning up bash.bashrc
  You may need to exit your terminal or reboot before running 'gravity-sync' commands
∞ Creating Gravity Sync Directories
Cloning into '/etc/gravity-sync/.gs'...
remote: Enumerating objects: 2808, done.
remote: Counting objects: 100% (248/248), done.
remote: Compressing objects: 100% (148/148), done.
remote: Total 2808 (delta 178), reused 115 (delta 92), pack-reused 2560
Receiving objects: 100% (2808/2808), 621.41 KiB | 4.50 MiB/s, done.
Resolving deltas: 100% (1787/1787), done.
∞ Starting Gravity Sync Configuration
∞ Initializing Gravity Sync (4.0.7)
✓ Evaluating arguments: CONFIGURE
✓ Creating new gravity-sync.conf

  Welcome to the Gravity Sync Configuration Wizard
  Please read through https://github.com/vmstan/gravity-sync/wiki before you continue
  Make sure that Pi-hole is running on this system before your configure Gravity Sync

» Gravity Sync Remote Host Settings
› Remote Pi-hole host address
? IP: ^C
```

# Funcionamiento

Una vez se ha instalado y configurado Gravity Sync en cada servidor, se puede proceder a probar el funcionamiento de forma manual antes de programar su funcionamiento automático.

## Comparación de la configuración

Desde el nodo autoritativo `pihole1` se ejecuta el comando `gravity-sync` para comparar la configuración de cada servidor Pi-hole:

```Bash
gravity-sync compare
```

Gravity Sync utilizará la conexión SSH al servidor `pihole2` para comparar las diferentes bases de datos de Pi-hole e indicará si hay diferencias:

```
root@pihole1 ~# gravity-sync compare
∞ Initializing Gravity Sync (4.0.7)
✓ Loading gravity-sync.conf
✓ Detecting local Pi-hole installation
∞ Detecting remote Pi-hole installation
✓ Detecting remote Pi-hole installation
∞ Checking on peer
✓ Gravity Sync remote peer is configured
✓ Evaluating arguments: COMPARE
» Remote target root@192.168.1.82
✓ Validating pathways to Pi-hole
✓ Validating pathways to DNSMASQ
∞ Hashing the remote Gravity Database
✓ Hashing the remote Gravity Database
✓ Comparing to the local Gravity Database
! Differences detected in the Gravity Database

∞ Hashing the remote DNS Records
✓ Hashing the remote DNS Records
✓ Comparing to the local DNS Records
! Differences detected in the DNS Records

! DNS CNAMEs not detected on the local Pi-hole

! Static DHCP Addresses not detected on the local Pi-hole
! Replication of Pi-hole settings is required
∞ Gravity Sync COMPARE completed after 2 seconds
```

{: .box-note}
La comparación de las bases de datos se realiza calculando el _hash_ de las mismas para saber si se han modificado.

## Sincronización mediante `push`

A continuación se realiza envío de la configuración del servidor `pihole1` al servidor remoto `pihole2` usando el parámetro **_push_** de Gravity Sync:

```Bash
gravity-sync push
```

A continuación se muestra el resultado de ese comando donde se ve la conexión mediante SSH, la comparación del _hash_ de las bases de datos de Pi-hole y, finalmente, la realización de una copia de seguridad de las mismas antes de actualizarlas:

```
root@pihole1 ~# gravity-sync push
∞ Initializing Gravity Sync (4.0.7)
✓ Loading gravity-sync.conf
✓ Detecting local Pi-hole installation
∞ Detecting remote Pi-hole installation
✓ Detecting remote Pi-hole installation
∞ Checking on peer
✓ Gravity Sync remote peer is configured
✓ Evaluating arguments: PUSH
» Remote target root@192.168.1.82
✓ Validating pathways to Pi-hole
✓ Validating pathways to DNSMASQ
∞ Hashing the remote Gravity Database
✓ Hashing the remote Gravity Database
✓ Comparing to the local Gravity Database
! Differences detected in the Gravity Database

∞ Hashing the remote DNS Records
✓ Hashing the remote DNS Records
✓ Comparing to the local DNS Records
! Differences detected in the DNS Records

! DNS CNAMEs not detected on the local Pi-hole

! Static DHCP Addresses not detected on the local Pi-hole
! Replication of Pi-hole settings is required
∞ Performing backup of remote Gravity Database
✓ Performing backup of remote Gravity Database
✓ Performing backup of local Gravity Database
✓ Checking Gravity Database copy integrity
✓ Pushing the local Gravity Database
∞ Setting file ownership on Gravity Database
✓ Setting file ownership on Gravity Database
∞ Setting file permissions on Gravity Database
✓ Setting file permissions on Gravity Database
∞ Performing backup of remote DNS Records
✓ Performing backup of remote DNS Records
✓ Performing backup of local DNS Records
✓ Pushing the local DNS Records
∞ Setting file ownership on DNS Records
✓ Setting file ownership on DNS Records
∞ Setting file permissions on DNS Records
✓ Setting file permissions on DNS Records
∞ Updating remote FTLDNS configuration
✓ Updating remote FTLDNS configuration
∞ Reloading remote FTLDNS services
✓ Reloading remote FTLDNS services
› Performing replicator diagnostics
∞ Rehashing the remote Gravity Database
✓ Rehashing the remote Gravity Database
✓ Recomparing to local Gravity Database

∞ Rehashing the remote DNS Records
✓ Rehashing the remote DNS Records
✓ Recomparing to local DNS Records

! DNS CNAMEs not detected on the local Pi-hole

! Static DHCP Addresses not detected on the local Pi-hole
✓ Saving updated data hashes
✓ Sending hashes to Gravity Sync peer
∞ Setting permissions on remote hashing files
✓ Setting permissions on remote hashing files
✓ Logging successful PUSH
∞ Gravity Sync PUSH completed after 13 seconds
```

## Sincronización automática

La configuración de la sincronización automática se realiza usando el [parámetro `auto quad`](https://github.com/vmstan/gravity-sync/wiki/Automation){:target="_blank"} para indicar que se quiere hacer aproximadamente cada 15 minutos:

```Bash
gravity-sync auto quad
```

Se puede conocer el estado de los servicios implicados en esta sincronización mediante el comando `systemctl`:

```
root@pihole1 ~# systemctl status gravity-sync.service
* gravity-sync.service - Synchronize Pi-hole instances
     Loaded: loaded (/etc/systemd/system/gravity-sync.service; disabled; vendor preset: enabled)
     Active: inactive (dead) since Sat 2024-04-06 15:14:17 CEST; 1min 0s ago
TriggeredBy: * gravity-sync.timer
    Process: 103042 ExecStart=/usr/local/bin/gravity-sync (code=exited, status=0/SUCCESS)
   Main PID: 103042 (code=exited, status=0/SUCCESS)
        CPU: 130ms

Apr 06 15:14:16 pihole1 gravity-sync[103042]: [40B blob data]
Apr 06 15:14:16 pihole1 gravity-sync[103042]: [87B blob data]
Apr 06 15:14:17 pihole1 gravity-sync[103042]: <E2><88><9E> Hashing the remote DNS Records
Apr 06 15:14:17 pihole1 gravity-sync[103042]: [35B blob data]
Apr 06 15:14:17 pihole1 gravity-sync[103042]: [77B blob data]
Apr 06 15:14:17 pihole1 gravity-sync[103042]: ! DNS CNAMEs not detected on the local Pi-hole
Apr 06 15:14:17 pihole1 gravity-sync[103042]: ! Static DHCP Addresses not detected on the local Pi-hole
Apr 06 15:14:17 pihole1 gravity-sync[103042]: ! No replication is required at this time
Apr 06 15:14:17 pihole1 gravity-sync[103042]: <E2><88><9E> Gravity Sync SMART exited after 2 seconds
Apr 06 15:14:17 pihole1 systemd[1]: gravity-sync.service: Succeeded.
```
```
root@pihole1 ~# systemctl status gravity-sync.timer
* gravity-sync.timer - Synchronize Pi-hole instances
     Loaded: loaded (/etc/systemd/system/gravity-sync.timer; enabled; vendor preset: enabled)
     Active: active (waiting) since Sat 2024-04-06 15:14:15 CEST; 1min 5s ago
    Trigger: Sat 2024-04-06 15:30:46 CEST; 15min left
   Triggers: * gravity-sync.service

Apr 06 15:14:15 pihole1 systemd[1]: Started Synchronize Pi-hole instances.
```

# Actualización

La comprobación de la versión instalada de Gravity Sync se realiza mediante el parámetro `version`:

```
root@pihole1 ~# gravity-sync version
∞ Initializing Gravity Sync (4.0.7)
✓ Evaluating arguments: VERSION
» Running version: 4.0.7
» Latest version: 4.0.7
∞ Gravity Sync VERSION exited after 0 seconds
```

Si hubiese una nueva versión, se puede utilizar el [parámetro `update` para actualizarla](https://github.com/vmstan/gravity-sync/wiki/Updating){:target="_blank"}.

# Referencias

* [High Availability Pi-Hole? Yes please!](https://technotim.live/posts/ha-pi-hold-gravity-sync/) @ TechnoTim

### Historial de cambios

* **2024-04-06**: Documento inicial
