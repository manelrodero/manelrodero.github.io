---
layout : post
blog-width: true
title: '¿Cómo se actualiza un LXC de TurnKey Core?'
date: '2024-04-07 15:45:16'
#last-updated: '2024-04-07 15:45:16'
published: true
tags:
- HomeLab
- Proxmox
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2024-04-07_cover.png"
thumbnail-img: ""
---

Desde que comencé a utilizar LXC de TurnKey Core en Proxmox hace 6 meses, siempre me ha preocupado la seguridad de estos contenedores y estaba con la mosca detrás de la oreja porque al ejecutar `apt upgrade` nunca había actualizaciones.

# TurnKey ~ Debian

Las _appliances_ de [TurnKey GNU/Linux](https://www.turnkeylinux.org/){:target=_blank} son imágenes construidas utilizando los binarios sin modificar de los repositorios oficiales de [Debian](https://www.debian.org/){:target=_blank}.

{: .box-note}
TurnKey Core LXC es una imagen Debian ya preinstalada, es decir, no hay que instalar el sistema operativo mediante el asistente como pasa al utilizar la [ISO Live](https://github.com/turnkeylinux/di-live){:target=_blank}.

La correspondencia entre las versiones de TurnKey y Debian se muestran en la siguiente tabla:

| Versión TurnKey	| Codename Debian/TurnKey | Versión Debian |
| -- | -- | -- |
| 14.x | Jessie	| 8.x |
| 15.x | Stretch | 9.x |
| 16.x | Buster | 10.x |
| 17.x | Bullseye | 11.x |
| 18.x | Bookworm	| 12.x |
| 19.x (future release)	| Trixie | 13.x |

Se puede conocer la versión que se está ejecutado mediante el comando `turnkey-version`:

```
root@lxc ~# turnkey-version
turnkey-core-17.1-bullseye-amd64
```

Y su correspondencia en Debian mediante el comando `lsb_release --all`:

```
root@lxc ~# lsb_release --all
No LSB modules are available.
Distributor ID: Debian
Description:    Debian GNU/Linux 11 (bullseye)
Release:        11
Codename:       bullseye
```

# Actualizaciones de seguridad

TurnKey se construye a partir de la **rama estable** de Debian por lo que únicamente recibe [**actualizaciones de seguridad**](https://security-team.debian.org/){:target=_blank}.

TurnKey está configurada para [instalar automáticamnte estas actualizaciones de seguridad](https://www.turnkeylinux.org/docs/auto-secupdates){:target=_blank} la primera vez que despliega una _appliance_ y cada noche según la programación indicada en el fichero `/etc/cron.d/cron-apt`:

```
20 1 * * * root test -x /usr/sbin/cron-apt && /usr/sbin/cron-apt
```

{: .box-note}
Se puede utilizar el comando `turnkey-install-security-updates` para instalar las actualizaciones de seguridad de forma manual.

Cada ejecución automática guarda un registro en el fichero `/var/log/cron-apt/log`, comenzando por una línea que indica el comienzo de la misma:

```
root@lxc ~# grep -i "CRON-APT RUN" /var/log/cron-apt/log
CRON-APT RUN [/etc/cron-apt/config]: Sun Apr  7 01:20:01 CEST 2024
```

# Actualizaciones de paquetes

La actualización manual de paquetes se realiza usando los comandos habituales:

```Bash
apt update
apt upgrade
```

Se puede comprobar si un paquete está actualizado utilizando el comando `apt policy`. El siguiente ejemplo muestra que el paquete **samba** está actualizado a la versión 2:4.13.13 desde los repositorios de seguridad de Debian:

```

root@lxc ~# apt policy samba
samba:
  Installed: 2:4.13.13+dfsg-1~deb11u6
  Candidate: 2:4.13.13+dfsg-1~deb11u6
  Version table:
 *** 2:4.13.13+dfsg-1~deb11u6 500
        500 http://security.debian.org bullseye-security/main amd64 Packages
        100 /var/lib/dpkg/status
     2:4.13.13+dfsg-1~deb11u5 500
        500 http://deb.debian.org/debian bullseye/main amd64 Packages
```

Según el [repositorio del paquete Samba para Debian Bullseye](https://packages.debian.org/bullseye/samba), la versión `2:4.13.13+dfsg-1~deb11u6` es la última actualización de seguridad a fecha de hoy, 7 de abril de 2024:

```
Package: samba (2:4.13.13+dfsg-1~deb11u6) [security]
SMB/CIFS file, print, and login server for Unix
```

{: .box-note}
Nótese que la versión de Samba depende de la versión de Debian. Por ejemplo, en el [repositorio del paquete Samba para Debian Bookworm](https://packages.debian.org/stable/samba), la versión más actual a día de hoy es la `2:4.17.12+dfsg-0+deb12u1`.

# Referencias

### Historial de cambios

* **2024-04-07**: Documento inicial
