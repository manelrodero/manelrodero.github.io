---
layout : post
blog-width: true
title: 'Instalar Vaultwarden en Proxmox LXC'
date: '2023-11-23 18:12:19'
published: true
tags:
- Proxmox
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2023-11-23_cover.png"
thumbnail-img: ""
---

Para seguir el proceso de migración de mi _Home Lab_ desde una pequeña [Raspberry Pi 4](instalar-raspberry-pi-os-64bits) a un [OptiPlex 7050 ejecutando Proxmox](proxmox-ve-802-en-un-dell-optiplex-7050) ahora le toca el turno a [**Vaultwarden**](https://github.com/dani-garcia/vaultwarden){:target="_blank"}, una implementación no oficial y mucho más ligera del servidor [Bitwarden](https://github.com/bitwarden/server){:target="_blank"} que se puede instalar localmente y es compatible con los [clientes oficiales](https://bitwarden.com/download/){:target="_blank"}.

En esta ocasion, la [instalación de Vaultwarden en Docker](instalacion-de-vaultwarden-en-docker) se realizará en un contenedor ligero LXC de Proxmox con TurnKey Core. El procedimiento para crear este LXC es el mismo que usé para [ejecutar Docker en Proxmox LXC](docker-en-proxmox-lxc-con-turnkey-core).

# Características del LXC

Este **gestor de contraseñas** con [Vaultwarden](https://github.com/dani-garcia/vaultwarden){:target="_blank"} se ejecutará sobre un LXC con las siguientes características:

* LXC no privilegiado, **con** _nesting_ y `keyctl`
* CT ID: **306**
* Hostname: **vaultwarden**
* Password + SSH key file
* Disco: 4GB en `local_lvm`
* CPU: 1 _core_
* Memoria: 512MB
* IP estática: **192.168.1.84/24**
* Gateway: **192.168.1.1**
* Dominio: **home**
* DNS: **192.168.1.81 192.168.1.82**

# Configuración del LXC

A continuación se pone en marcha el LXC desde la CLI de Proxmox y se procede a configurarlo de la misma manera que ya expliqué en el artículo sobre [LXC y Docker](docker-en-proxmox-lxc-con-turnkey-core):

```
pct start 306
```

Se puede acceder al LXC utilizando la opción `Console` de la GUI o, mucho mejor, mediante **SSH** gracias a la configuración de las claves públicas que se hizo en el mismo:

```Bash
ssh root@192.168.1.84
```

La configuración inicial es la habitual en una plantilla LXC de TurnKey Core:

* First Boot Configuration: `Skip` / `Skip` / `Install`
* Advanced Menu: `Quit`
* Zona horaria **Europe/Madrid** con `dpkg-reconfigure tzdata`
* Actualizar con `apt update && apt upgrade -y`
* Reiniciar con `reboot`

# Instalación de Docker

La instalación de Docker es idéntica a la que expliqué en el artículo "[Docker en Proxmox LXC con TurnKey Core](docker-en-proxmox-lxc-con-turnkey-core)":

* Añadir la clave oficial GPG de Docker
* Añadir el repositorio a las fuentes de `apt`
* Instalar los paquetes de Docker: `docker-ce`, `docker-ce-cli`, etc.

# Instalación de Nginx Proxy Manager

Una vez se ha instalado Docker, y dado que Vaultwarden requiere un acceso HTTPS, se puede instalar **Nginx Proxy Manager** de la misma forma que lo hice en la Raspberry Pi 4 (ver artículo "[Instalación de Nginx Proxy Manager en Docker](instalacion-de-nginx-proxy-manager-en-docker)"):

* Crear el fichero `docker-compose.yml`
* Ponerlo en marcha usando `docker compose up -d`
* Realizar la configuración inicial
* Crear un registro de tipo A en Cloudflare
* Configurar la petición de certificados SSL en Let's Encrypt para ese dominio
* Configurar un proxy host para ese dominio dirigido al docker de Vaultwarden

# Instalación de Vaultwarden

La instalación de **Vaultwarden** también se puede realizar igual que en la Raspberry Pi 4 (ver artículo "[Instalación de Vaultwarden en Docker](instalacion-de-vaultwarden-en-docker)"):

* Crear el fichero `docker-compose.yml`
* Ponerlo en marcha usando `docker compose up -d`
* Realizar la configuración inicial
* Habilitar el Admin Panel
* Configurar los parámetros de seguridad

{: .box-note}
**Nota**: En mi caso, al estar realizando una migración, he copiado el volúmen `data` desde la Raspberry Pi 4 antes de poner en marcha el Docker. De esta manera recupero la misma configuración, la base de datos, la _caché_ de iconos, etc.

```
drwxr-xr-x 2 root root    4096 Dec 18  2022 attachments
-rw-r--r-- 1 root root    1432 Dec 18  2022 config.json
-rw-r--r-- 1 root root  651264 Nov 12 13:10 db.sqlite3
-rw-r--r-- 1 root root   32768 Nov 23 19:25 db.sqlite3-shm
-rw-r--r-- 1 root root 4128272 Nov 23 19:25 db.sqlite3-wal
drwxr-xr-x 2 root root   16384 Nov 23 19:24 icon_cache
-rw-r--r-- 1 root root    1679 Dec 18  2022 rsa_key.pem
-rw-r--r-- 1 root root     451 Dec 18  2022 rsa_key.pub.pem
drwxr-xr-x 2 root root    4096 Dec 18  2022 sends
drwxr-xr-x 2 root root    4096 Dec 18  2022 tmp
```

Al iniciar sesión por primera vez, se muestra un mensaje indicando que se requiere [incrementar el número de iteraciones KDF](https://bitwarden.com/help/what-encryption-is-used/#changing-kdf-iterations){:target="_blank"}:

![KDF Settings][1]

La interfaz web [recomienda un valor de 600000](https://bitwarden.com/help/kdf-algorithms/){:target="_blank"} para ser compatible con FIPS-140, pero la documentación indica que es recomendable realizar incrementos de 100000 y comprobar si todos los dispositivos que usemos son capaces de abrir y desbloquear el _vault_ de forma fluída.

{: .box-note}
**Nota**: En mi caso, la configuración recuperada de una versión 1.27.0 tenía un valor por defecto de 100000, bastante diferente del valor por defecto de 600000 de la versión 1.30.0 que se acaba de instalar.

Al acceder al **Panel de Administración** se muestra un mensaje indicando que es más seguro utilizar un _string_ seguro **Argon 2 PHC** (disponible desde la versión [1.28.0+](https://github.com/dani-garcia/vaultwarden/releases/tag/1.28.0){:target="_blank"}) en lugar de un texto plano para el `ADMIN_TOKEN`:

![Secure Argon2 PHC string][2]

Para generarlo se utiliza el comando `vaultwarden hash --preset owasp` desde el propio Docker:

```
root@vaultwarden ~# docker exec -it vaultwarden /vaultwarden hash --preset owasp
Generate an Argon2id PHC string using the 'owasp' preset:

Password: 
Confirm Password: 

ADMIN_TOKEN='$argon2id$v=19$m=19456,t=2,p=1$7LDGBu/bVVcKsiRvYT1F4ikMPNGrJnR+43GfS/5w4BI$QydzrtxEcvqc4g2v6gXpAQKacRmlYdxFARzT90ScUfI'

Generation of the Argon2id PHC string took: 36.537429ms
```

Una vez generado, se cambia en los `General Settings` y se graba la configuración. Si todo funciona correctamente, se podrá iniciar sesión utilizando el _password_ usado para generar el _string_ seguro.

[1]: /assets/img/blog/2023-11-23_image_1.png "KDF Settings"
[2]: /assets/img/blog/2023-11-23_image_2.png "Secure Argon2 PHC string"
