---
layout : post
blog-width: true
title: 'Plantilla de LXC con Debian 12 para ejecutar Docker en Proxmox'
date: '2025-08-12 20:14:43'
last-updated: '2025-08-13 16:40:00'
published: true
tags:
- Proxmox
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2025-08-12_cover.png"
thumbnail-img: ""
---

En este artículo explicaré cómo crear una plantilla de **contenedor Linux (LXC) en Proxmox**, que esté basada en **Debian 12 Standard** y que permita ejecutar **contenedores Docker**.

{: .box-note}
**Nota**: El soporte oficial de Proxmox recomienda ejecutar Docker en máquinas virtuales, pero un contenedor Linux (LXC) permite hacerlo con menos recursos del hipervisor y con velocidades de arranque mucho más rápidas.

Este artículo será bastante esquemático y utilizará información que ya he explicado en artículos similares usando **TurnKey Core** como sistema operativo:

* [Docker en Proxmox LXC con Debian 12](docker-en-proxmox-lxc-con-debian-12)
* [Docker en Proxmox LXC con Debian 11](docker-en-proxmox-lxc-con-turnkey-core)
* [Crear un LXC desde el CLI de Proxmox](crear-un-lxc-desde-el-cli-de-proxmox)

## Actualizar Proxmox

Antes de empezar, es recomendable asegurarse que Proxmox está correctamente actualizado:

```bash
pveversion
# pve-manager/8.4.6/c5b55b84d1f84ea6 (running kernel: 6.8.12-13-pve)
pveupdate
echo y | pveupgrade
apt autoremove -y
reboot -h now
pveversion
# pve-manager/8.4.11/14a32011146091ed (running kernel: 6.8.12-13-pve)
```

## Descargar plantilla Debian 12

Antes de poder crear un contenedor Linux (LXC) es necesario descargra la plantilla del SO que se quiere ejecutar:

```bash
pveam update
pveam available --section system | grep -i debian
pveam download local debian-12-standard_12.7-1_amd64.tar.zst
```

## Crear el contenedor base

Para crear el contenedor se ejecutará el comando `pct create` desde la CLI de Proxmox.

Para hacerlo más flexible y seguro, se pregunta el `CT ID` y el `password` antes de ejecutarlo:

```bash
while true; do
  read -p "Introduce el CT ID: " ct_id
  if pct list | awk '{print $1}' | grep -qw "$ct_id"; then
    echo "Error: El CT ID $ct_id ya existe. Por favor, introduce un CT ID diferente."
  else
    break
  fi
done

while true; do
  read -s -p "Introduce el password del usuario 'root': " password1
  echo
  read -s -p "Confirma el password del usuario 'root': " password2
  echo
  if [ "$password1" == "$password2" ]; then
    password=$password1
    break
  else
    echo "Error: Los passwords no coinciden. Por favor, inténtalo de nuevo."
  fi
done

pct create "$ct_id" local:vztmpl/debian-12-standard_12.7-1_amd64.tar.zst \
  --ostype debian --arch amd64 \
  --hostname debian12-lxc --unprivileged 1 \
  --password "$password" --ssh-public-keys /root/id_edcsa.pub \
  --storage local-lvm --rootfs local-lvm:4 \
  --cores 1 \
  --memory 512 --swap 512 \
  --net0 name=eth0,bridge=vmbr0,firewall=1,ip=192.168.1.200/24,gw=192.168.1.1 \
  --features nesting=1,keyctl=1 \
  --onboot 1 \
  --start 0

unset password
unset password1
unset password2
```

{: .box-note}
También se puede descargar mi script [`create_ct.sh`](https://gist.github.com/manelrodero/ab36b56690a8f7b565cdd5c22fd33476){:target="_blank"} y adaptar los valores por defecto antes de ejecutarlo.

## Poner en marcha el contenedor

Se pone en marcha el contenedor LXC usando el comando:

```bash
pct start "$ct_id"
```

## Iniciar sesión desde Proxmox

Se accede al contenedor LXC desde Proxmox, no hace falta hacerlo por SSH:

```bash
pct enter "$ct_id"
```

## Configurar `locale`

Las [variables de localización `locale`](las-variables-de-localizacion-locale-en-linux) son importantes para manejar algunos aspectos relacionados con la configuración regional (formatos de idioma, fechas, números, etc.).

Si no están bien configurados, se pueden generar errores similares a los siguientes al ejecutar determinados comandos:

```plaintext
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
     LANGUAGE = (unset),
     LC_ALL = (unset),
     LANG = "en_US.UTF-8"
  are supported and installed on your system.
perl: warning: Falling back to the standard locale ("C").
locale: Cannot set LC_CTYPE to default locale: No such file or directory
locale: Cannot set LC_MESSAGES to default locale: No such file or directory
locale: Cannot set LC_ALL to default locale: No such file or directory
```

Para solucionarlo, únicamente hay que **generar** los locales que se requieran, en este caso únicamente `en_US.UTF-8`:

```bash
sed -i 's/^# *\(en_US.UTF-8 UTF-8\)/\1/' /etc/locale.gen
locale-gen
```

## Configurar la zona horaria

Debian 12 tiene configurada por defecto la zona horaria `Etc/UTC`, tal como se puede observar a continuación:

```bash
date && date -u
```

```plaintext
Wed Aug 13 02:38:23 PM UTC 2025
Wed Aug 13 02:38:23 PM UTC 2025
```

Es recomendable cambiar la **zona horaria** a la nuestra (en mi caso es `Europe/Madrid`) ejecutando los sigiuentes comandos:

```bash
ln -sf /usr/share/zoneinfo/Europe/Madrid /etc/localtime
echo "Europe/Madrid" > /etc/timezone
dpkg-reconfigure -f noninteractive tzdata
```

Después del cambio ya tendremos la hora correcta, incluyendo posibles cambios de invierno/verano:

```plaintext
Wed Aug 13 04:39:38 PM CEST 2025
Wed Aug 13 02:39:38 PM UTC 2025
```

## Actualizar repositorios

Para poder instalar los paquetes que requiere esta plantilla, es necesario actualizar los repositorios que incluye Debian 12 usando el comando `apt update`.

Como **Debian 13** (`Trixie`) es la nueva versión estable, el gestor de paquetes indica que **Debian 12** (`Bookworm`) es ahora **oldstable** para seguir recibiendo actualizaciones de seguridad y que se han publicado nuevas actualizaciones acumulativas **12.11**:

```plaintext
N: Repository 'http://security.debian.org bookworm-security InRelease' changed its 'Suite' value from 'stable-security' to 'oldstable-security'
N: Repository 'http://deb.debian.org/debian bookworm InRelease' changed its 'Version' value from '12.7' to '12.11'
N: Repository 'http://deb.debian.org/debian bookworm InRelease' changed its 'Suite' value from 'stable' to 'oldstable'
N: Repository 'http://deb.debian.org/debian bookworm-updates InRelease' changed its 'Suite' value from 'stable-updates' to 'oldstable-updates'
```

Se pueden aceptar los nuevos metadatos del repositorio (`Suite` y `Version`) ejecutando el siguiente comando:

```bash
apt update --allow-releaseinfo-change
```

## Actualizar paquetes

A continuación se procede a actualizar los paquetes de la manera habitual:

```bash
apt upgrade -y
```

## Instalar Docker Engine

La instalación de **Docker Engine** se realiza desde el repositorio oficial ejecutando los siguientes comandos:

```bash
# Add Docker's official GPG key:
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
apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

Se puede comprobar que todo funciona correctamente comprobando el servicio `docker` y ejecutando el contenedor `hello-world`:

```bash
systemctl status docker
docker run hello-world
```

## (Opcional) Usuario sin privilegios

Me gusta crear un usuario sin privilegios para ejecutar los contenedores Docker:

```bash
# Usuario sin privilegios (se usará en los siguientes comandos)
user="manel"

# Crear el usuario $user
useradd -m -u 1000 -U -G docker -s /bin/bash "$user"
passwd "$user"
cat /etc/passwd | grep -i "$user"
cat /etc/group | grep -i "$user"
```

Para que este usuario también pueda entrar por SSH mediante clave, se copia el mismo fichero `authorized_keys` del usuario `root`:

```bash
# Crear el directorio '.ssh' y copiar 'authorized_keys'
mkdir "/home/$user/.ssh"
cp ~/.ssh/authorized_keys "/home/$user/.ssh/"
chown -R "$user:$user" "/home/$user/.ssh"
```

Se crea la estructura necesaria para ejecutar de madrugada el _script_ [`~/dockers/backup_dockers.sh`](https://gist.github.com/manelrodero/a693616deacf2a04e93f9959a7a6ee2e){:target="_blank"} usando **cron**:

```bash
# Crear el directorio 'dockers' y descargar el fichero 'backup_dockers.sh'
mkdir "/home/$user/dockers"
curl -fsSL https://gist.githubusercontent.com/manelrodero/a693616deacf2a04e93f9959a7a6ee2e/raw/74c28ea97357c7e8366ce0ab01dbfa9c5360a508/backup_dockers.sh -o "/home/$user/dockers/backup_dockers.sh"
chmod +x "/home/$user/dockers/backup_dockers.sh"
chown -R "$user:$user" "/home/$user/dockers"

# Programar la ejecución del script 'backup_dockers.sh'
echo "# m h  dom mon dow   command" > "/tmp/${user}_cron"
echo "00 2 * * * /home/$user/dockers/backup_dockers.sh >/dev/null 2>&1" >> "/tmp/${user}_cron"
crontab -u "$user" "/tmp/${user}_cron"
crontab -u "$user" -l
rm "/tmp/${user}_cron"
```

## (Opcional) Directorio para `rsync`

En mi caso, me gusta que los LXC monten en el directorio `/mnt/rsync` un directorio del _host_ Proxmox para poder realizar copias de seguridad fuera del contenedor.

```bash
mkdir /mnt/rsync
chown root:"$user" /mnt/rsync
chmod 770 /mnt/rsync
```

## (Opcional) Instalación y configuración de `sudo`

Para que el usuario sin privilegios no tenga problemas a la hora de copiar cualquier fichero de los Docker, será necesario que ejecute el comando `rsync` usando `sudo`.

Para ello será necesario instalar el paquete e indicar qué comandos se le permiten ejecutar (`rsync`, `nano`, `rm` y `ls`):

```bash
apt install sudo -y
echo "# Configuración Sudo para $user" > "/etc/sudoers.d/$user"
echo "$user ALL=(ALL) NOPASSWD: /usr/bin/rsync" >> "/etc/sudoers.d/$user"
echo "$user ALL=(ALL) NOPASSWD: /usr/bin/nano" >> "/etc/sudoers.d/$user"
echo "$user ALL=(ALL) NOPASSWD: /usr/bin/rm" >> "/etc/sudoers.d/$user"
echo "$user ALL=(ALL) NOPASSWD: /usr/bin/ls" >> "/etc/sudoers.d/$user"
chmod 0440 "/etc/sudoers.d/$user"
visudo -cf "/etc/sudoers.d/$user"
```

## Instalar `unattended-upgrades`

Una de las cosas que más me gustaban de TurnKey Core es que [instalaba automáticamente las actualizaciones de seguridad](como-se-actualiza-un-lxc-de-turnkey-core).

En esta plantilla usando Debian 12 se hará lo mismo instalando `unattended-upgrades`:

```bash
apt install -y unattended-upgrades apt-listchanges
```

En esta primera toma de contacto, se deja la configuración por defecto que utiliza los siguientes ficheros en `/etc/apt/apt.conf.d`:

| Archivo | Propósito |
| --- | --- |
| `01autoremove` | Configura la eliminación automática de paquetes no usados |
| `20auto-upgrades` | Controla la ejecución automática de actualizaciones |
| `20listchanges` | Muestra cambios de paquetes durante actualizaciones |
| `50unattended-upgrades` | Define qué actualizaciones se instalan automáticamente |
| `70debconf` | Configuración generada por debconf (no relevante aquí) |

Se puede ver si un paquete está actualizado usando el comando `apt policy`. Por ejemplo, la versión de `curl` instalada actualmente es la más reciente:

```bash
apt policy curl
```

```plaintext
curl:
  Installed: 7.88.1-10+deb12u12
  Candidate: 7.88.1-10+deb12u12
  Version table:
 *** 7.88.1-10+deb12u12 500
        500 http://deb.debian.org/debian bookworm/main amd64 Packages
        100 /var/lib/dpkg/status
     7.88.1-10+deb12u5 500
        500 http://security.debian.org bookworm-security/main amd64 Packages
```

Se puede comprobar el funcionamiento de las actualizaciones automáticas ejecutando el comando:

```bash
unattended-upgrades -d
```

## Convertir en plantilla para nuevos CT

Una vez configurado el contenedor LXC, se usa el comando `exit` para abonadarlo y volver a Proxmox.

Desde allí se para el contenedor y se convierte en plantilla mediante los siguientes comandos:

```bash
pct stop "$ct_id"
pct template "$ct_id"
```

## Crear un nuevo CT clonando la plantilla

Al clonar la plantilla usando la opción **full clone** se crea un nuevo CT totalmente independiente:

```bash
# Nombre del LXC (se usará en los siguientes comandos)
lxcname="lxc-test"

while true; do
  read -p "Introduce el nuevo CT ID: " new_ct_id
  if pct list | awk '{print $1}' | grep -qw "$new_ct_id"; then
    echo "Error: El nuevo CT ID $ct_id ya existe. Por favor, introduce un CT ID diferente."
  else
    break
  fi
done

# Clonar la plantilla "$ct_id" en el LXC 901
pct clone "$ct_id" "$new_ct_id" --hostname "$lxcname" --full
```

El proceso de clonación de un LXC tarda unos pocos segundos.

## (Opcional) Añadir el **mountpoint** para los _backups_

```bash
mkdir "/backups/rsync/$lxcname"
chown 100000:101000 "/backups/rsync/$lxcname"
chmod 770 "/backups/rsync/$lxcname"
pct set "$new_ct_id" -mp0 "/backups/rsync/$lxcname,mp=/mnt/rsync"
```

## Cambiar la dirección IP

En este ejemplo se utilizará una IP no utilizada en mi entorno, por ejemplo `192.168.1.250`:

```bash
pct set "$new_ct_id" -net0 name=eth0,bridge=vmbr0,ip=192.168.1.250/24,gw=192.168.1.1
```

## (Opcional) Cambiar los parámetros hardware

```bash
pct set "$new_ct_id" -cores 2 -memory 1024 -swap 512
```

## Iniciar el contenedor

```bash
pct start "$new_ct_id"
```

## Iniciar sesión por SSH mediante claves

Si las claves SSH se han configurado correctamente, se podrá iniciar sesión sin necesidad de contraseña usando el comando:

```bash
ssh manel@192.168.1.250
```

A partir de aquí ya solo queda "jugar" con este LXC para instalar la aplicación que se necesite ;-)

### Historial de cambios

* **2025-08-12**: Documento inicial
* **2025-08-13**: Simplificación y automatización de comandos
