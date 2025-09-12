---
layout : post
blog-width: true
title: 'Plantilla de LXC con Debian 13 para ejecutar Docker en Proxmox'
date: '2025-09-10 19:21:34'
#last-updated: '2025-09-10 19:21:34'
published: true
tags:
- Proxmox
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2025-09-10_cover.png"
thumbnail-img: ""
---

En este artículo explicaré cómo crear una plantilla de **contenedor Linux (LXC) en Proxmox**, que esté basada en **Debian 13 Standard** y que permita ejecutar **contenedores Docker**.

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
# pve-manager/8.4.11/14a32011146091ed (running kernel: 6.8.12-13-pve)
pveupdate
echo y | pveupgrade
apt autoremove -y
reboot -h now
pveversion
# pve-manager/8.4.13/5b08ebc2823dd9cb (running kernel: 6.8.12-14-pve)
```

## Descargar plantilla Debian 13

Antes de poder crear un contenedor Linux (LXC) es necesario descargra la plantilla del SO que se quiere ejecutar:

```bash
pveam update
pveam available --section system | grep -i debian
pveam download local debian-13-standard_13.1-1_amd64.tar.zst
```

## Crear el contenedor base

Para crear el contenedor se usa el comando `pct create` desde la CLI de Proxmox, preguntando antes una serie de parámetros imprescindibles (el `ID` del contenedor, la contraseña del usuario `root`, la dirección IP y la puerta de enlace):

```bash
clear

RED='\033[1;31m'
GREEN='\033[1;32m'
CYAN='\033[1;36m'
NC='\033[0m'

echo -e "${GREEN}Introduce el nombre de la plantilla:${NC}"
read -p "> " lxcname

while true; do
  echo -e "${GREEN}Introduce el CT ID:${NC}"
  read -p "> " ct_id
  if pct list | awk '{print $1}' | grep -qw "$ct_id"; then
    echo -e "${RED}Error: El CT ID $ct_id ya existe.${NC}"
  else
    break
  fi
done

while true; do
  echo -e "${GREEN}Introduce el password de 'root':${NC}"
  read -s -p "> " password1
  echo
  echo -e "${GREEN}Confirma el password de 'root':${NC}"
  read -s -p "> " password2
  echo
  if [ "$password1" == "$password2" ]; then
    password=$password1
    break
  else
    echo -e "${RED}Error: Los passwords no coinciden.${NC}"
  fi
done

valid_ip() {
  [[ "$1" =~ ^([0-9]{1,3}\.){3}[0-9]{1,3}$ ]] || return 1
  IFS='.' read -r o1 o2 o3 o4 <<< "$1"
  for octet in "$o1" "$o2" "$o3" "$o4"; do
    (( octet >= 0 && octet <= 255 )) || return 1
  done
  return 0
}

while true; do
  echo -e "${GREEN}Introduce la dirección IP:${NC}"
  read -p "> " ip
  if valid_ip "$ip"; then
    ip="$ip/24"
    break
  else
    echo -e "${RED}Formato de IP inválido.${NC}"
  fi
done

while true; do
  echo -e "${GREEN}Introduce la puerta de enlace:${NC}"
  read -p "> " gw
  if valid_ip "$gw"; then
    break
  else
    echo -e "${RED}Formato de IP inválido.${NC}"
  fi
done

echo -e "${CYAN}Creando el contenedor ${ct_id}...${NC}"
pct create "$ct_id" local:vztmpl/debian-13-standard_13.1-1_amd64.tar.zst \
  --ostype debian --arch amd64 \
  --hostname "$lxcname" --unprivileged 1 \
  --password "$password" --ssh-public-keys /root/id_edcsa.pub \
  --storage local-lvm --rootfs local-lvm:2 \
  --cores 1 \
  --memory 512 --swap 512 \
  --net0 name=eth0,bridge=vmbr0,firewall=1,ip="$ip",gw="$gw" \
  --features nesting=1,keyctl=1 \
  --onboot 1 \
  --start 0
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
sed -i 's/^# *\(es_ES.UTF-8 UTF-8\)/\1/' /etc/locale.gen
locale-gen
cat <<EOF > /etc/default/locale
LANG=en_US.UTF-8
LANGUAGE=en_US:en
LC_CTYPE=es_ES.UTF-8
EOF
```

## Configurar la zona horaria

Debian tiene configurada por defecto la zona horaria `Etc/UTC`, tal como se puede observar a continuación:

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

## Configurar `.bashrc`

```bash
# Forzar colores en PROMPT y grep
sed -i 's/^#force_color_prompt=yes/force_color_prompt=yes/' /etc/skel/.bashrc
sed -i -E "s/^([[:space:]]*)#([[:space:]]*)alias grep='grep --color=auto'/\1\2alias grep='grep --color=auto'/" /etc/skel/.bashrc

# Aplicar también al usuario root
cp ~/.bashrc ~/.bashrc.bak
cp ~/.profile ~/.profile.bak
cp /etc/skel/.bashrc ~/.bashrc
cp /etc/skel/.profile ~/.profile

# Logout per inactivitat
cat <<EOF >> /etc/bash.bashrc

# Logout per inactivitat 30 minuts
export TMOUT=1800
readonly TMOUT
EOF
```

## Actualizar repositorios

Como **Debian 13** (`Trixie`) es la nueva versión estable, únicamente es necesario ejecutar el comando habitual para actualizar los repositorios:

```bash
apt update
```

## Instalar Docker Engine

La instalación de **Docker Engine** se realiza desde el repositorio oficial ejecutando los siguientes comandos:

```bash
# Add Docker's official GPG key:
apt install ca-certificates curl -y
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
clear

RED='\033[1;31m'
GREEN='\033[1;32m'
CYAN='\033[1;36m'
NC='\033[0m'

echo -e "${GREEN}Introduce el nombre del usuario sin privilegios:${NC}"
read -p "> " user

echo -e "${CYAN}Creando el usuario '${user}'...${NC}"
useradd -m -u 1000 -U -G docker -s /bin/bash "$user"

echo -e "${CYAN}Estableciendo contraseña para '${user}'...${NC}"
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
curl -fsSL https://gist.githubusercontent.com/manelrodero/a693616deacf2a04e93f9959a7a6ee2e/raw/d3b7e727690e0189334136f66ad1752606c3245e/backup_dockers.sh -o "/home/$user/dockers/backup_dockers.sh"
chmod +x "/home/$user/dockers/backup_dockers.sh"
chown -R "$user:$user" "/home/$user/dockers"

# Programar la ejecución del script 'backup_dockers.sh'
echo "# m h  dom mon dow   command" > "/tmp/${user}_cron"
echo "00 2 * * * /home/$user/dockers/backup_dockers.sh >/dev/null 2>&1" >> "/tmp/${user}_cron"
crontab -u "$user" "/tmp/${user}_cron"
crontab -u "$user" -l
rm "/tmp/${user}_cron"
```

## (Opcional) Configuración para `rsync`

En mi caso, me gusta que los LXC monten en el directorio `/mnt/rsync` un directorio del _host_ Proxmox para poder realizar copias de seguridad fuera del contenedor.

```bash
mkdir /mnt/rsync
chown root:"$user" /mnt/rsync
chmod 770 /mnt/rsync
apt install rsync -y
```

## (Opcional) Instalación y configuración de `sudo`

Para que el usuario sin privilegios no tenga problemas a la hora de copiar cualquier fichero de los Docker, será necesario que ejecute el comando `rsync` usando `sudo`.

Para ello será necesario instalar el paquete e indicar qué comandos se le permiten ejecutar (`rsync`, `nano`, `rm`, `ls`, `reboot` y `shutdown`):

```bash
apt install sudo -y
echo "# Configuración Sudo para $user" > "/etc/sudoers.d/$user"
echo "$user ALL=(ALL) NOPASSWD: /usr/bin/rsync" >> "/etc/sudoers.d/$user"
echo "$user ALL=(ALL) NOPASSWD: /usr/bin/nano" >> "/etc/sudoers.d/$user"
echo "$user ALL=(ALL) NOPASSWD: /usr/bin/rm" >> "/etc/sudoers.d/$user"
echo "$user ALL=(ALL) NOPASSWD: /usr/bin/ls" >> "/etc/sudoers.d/$user"
echo "$user ALL=(ALL) NOPASSWD: /usr/bin/cat" >> "/etc/sudoers.d/$user"
echo "$user ALL=(ALL) NOPASSWD: /usr/sbin/reboot" >> "/etc/sudoers.d/$user"
echo "$user ALL=(ALL) NOPASSWD: /usr/sbin/shutdown" >> "/etc/sudoers.d/$user"
chmod 0440 "/etc/sudoers.d/$user"
visudo -cf "/etc/sudoers.d/$user"
```

## (Opcional) Instalar `unattended-upgrades`

Una de las cosas que más me gustaban de TurnKey Core es que [instalaba automáticamente las actualizaciones de seguridad](como-se-actualiza-un-lxc-de-turnkey-core) sin tener que hacerlo manualmente.

En esta plantilla usando Debian 13 se conseguirá lo mismo instalando el paquete `unattended-upgrades`:

```bash
apt install -y unattended-upgrades apt-listchanges
```

La configuración se define en siguientes ficheros de `/etc/apt/apt.conf.d`:

| Archivo | Propósito |
| --- | --- |
| `01autoremove` | Configura la eliminación automática de paquetes no usados |
| `20auto-upgrades` | Controla la ejecución automática de actualizaciones |
| `20listchanges` | Muestra cambios de paquetes durante actualizaciones |
| `50unattended-upgrades` | Define qué actualizaciones se instalan automáticamente |
| `70debconf` | Configuración generada por debconf (no relevante aquí) |

Al principio del fichero `50unattended-upgrades` se definen qué actualizaciones se instalarán según el patrón `Origins-Pattern`:

```plaintext
Unattended-Upgrade::Origins-Pattern {
        // Codename based matching:
        // This will follow the migration of a release through different
        // archives (e.g. from testing to stable and later oldstable).
        // Software will be the latest available for the named release,
        // but the Debian release itself will not be automatically upgraded.
//      "origin=Debian,codename=${distro_codename}-updates";
//      "origin=Debian,codename=${distro_codename}-proposed-updates";
        "origin=Debian,codename=${distro_codename},label=Debian";
        "origin=Debian,codename=${distro_codename},label=Debian-Security";
        "origin=Debian,codename=${distro_codename}-security,label=Debian-Security";
//      "o=Debian Backports,n=${distro_codename}-backports,l=Debian Backports";
```

Según este patrón, inicialmente únicamente se actualizan los paquetes:

* `trixie` con etiqueta `Debian`
* `trixie` con etiqueta `Debian-Security`
* `trixie-security` con etiqueta `Debian-Security`

Para que también se instalen los paquetes `trixie-updates` será necesario descomentar la línea correspondiente del fichero mediante el siguiente comando:

```bash
sed -i -E 's|^([[:space:]]*)//([[:space:]]*)"origin=Debian,codename=\$\{distro_codename\}-updates";|\1  \2"origin=Debian,codename=${distro_codename}-updates";|' /etc/apt/apt.conf.d/50unattended-upgrades
```

Finalmente, se fuerza una ejecución manual de `unattended-upgrades` con el flag `-d` para depurar lo que hace por pantalla:

```bash
unattended-upgrades -d
```

Si todo funciona correctamente, cuando acabe el proceso todos los paquetes deberían estar actualizados:

```bash
apt update
```

```plaintext
Hit:1 http://deb.debian.org/debian trixie InRelease
Hit:2 http://deb.debian.org/debian trixie-updates InRelease                                     
Hit:3 http://security.debian.org trixie-security InRelease                                      
Hit:4 https://download.docker.com/linux/debian trixie InRelease           
All packages are up to date.
```

Para ver si un paquete está actualizado se puede usar el comando `apt policy`. Por ejemplo, la versión de `curl` instalada actualmente es la más reciente:

```bash
apt policy curl
```

```plaintext
curl:
  Installed: 8.14.1-2
  Candidate: 8.14.1-2
  Version table:
 *** 8.14.1-2 500
        500 http://deb.debian.org/debian trixie/main amd64 Packages
        100 /var/lib/dpkg/status
```

## Instalar paquetes adicionales

Se instalan algunos paquetes útiles para monitorizar el rendimiento del sistema y las conexiones de red:

```bash
apt install htop -y
apt install net-tools -y
```

El paquete `net-tools`, aunque bastante antiguo, se ha incluido para tener los comandos clásicos:

| Acción | net-tools | iproute2 |
| --- | --- | --- |
| Ver interfaces de red | `ifconfig` | `ip addr` o `ip link` |
| Ver rutas | `route -n` | `ip route` |
| Ver conexiones TCP/UDP | `netstat -tulnp` | `ss -tulnp` |
| Activar interfaz | `ifconfig eth0 up` | `ip link set eth0 up` |

## Limpieza del historial

Se limpia el historial (en memoria y el archivo persistente):

```bash
history -c
unset HISTFILE
rm -f ~/.bash_history
```

## Convertir en plantilla para nuevos CT

Una vez configurado, se puede abandonar el contenedor LXC y volver a Proxmox:

```bash
exit
```

Desde allí se para el contenedor y se convierte en plantilla mediante los siguientes comandos:

```bash
pct stop "$ct_id"
pct template "$ct_id"
```

## Crear un nuevo CT clonando la plantilla

Al clonar la plantilla usando la opción **full clone** se crea un nuevo CT totalmente independiente:

```bash
clear

RED='\033[1;31m'
GREEN='\033[1;32m'
CYAN='\033[1;36m'
NC='\033[0m'

echo -e "${GREEN}Introduce el nombre del nuevo contenedor:${NC}"
read -p "> " lxcname

while true; do
  echo -e "${GREEN}Introduce el CT ID de la plantilla:${NC}"
  read -p "> " ct_id

  if ! pct status "$ct_id" &>/dev/null; then
    echo -e "${RED}Error: El CT ID $ct_id no existe.${NC}"
    continue
  fi

  if ! pct config "$ct_id" | grep -q "^template: 1"; then
    echo -e "${RED}Error: El CT ID $ct_id no es una plantilla.${NC}"
    continue
  fi

  break
done

while true; do
  echo -e "${GREEN}Introduce el CT ID del nuevo contenedor:${NC}"
  read -p "> " new_ct_id

  if pct status "$new_ct_id" &>/dev/null; then
    echo -e "${RED}Error: El CT ID $new_ct_id ya existe.${NC}"
  else
    break
  fi
done

echo -e "${CYAN}Clonando la plantilla...${NC}"
pct clone "$ct_id" "$new_ct_id" --hostname "$lxcname" --full
```

El proceso de clonación de un LXC tarda unos pocos segundos.

## (Opcional) Añadir el **mountpoint** para los _backups_

```bash
mkdir "/backups/rsync/$new_ct_id-$lxcname"
chown 100000:101000 "/backups/rsync/$new_ct_id-$lxcname"
chmod 770 "/backups/rsync/$new_ct_id-$lxcname"
pct set "$new_ct_id" -mp0 "/backups/rsync/$new_ct_id-$lxcname,mp=/mnt/rsync"
```

## Cambiar la dirección IP

```bash
clear

valid_ip() {
  [[ "$1" =~ ^([0-9]{1,3}\.){3}[0-9]{1,3}$ ]] || return 1
  IFS='.' read -r o1 o2 o3 o4 <<< "$1"
  for octet in "$o1" "$o2" "$o3" "$o4"; do
    (( octet >= 0 && octet <= 255 )) || return 1
  done
  return 0
}

while true; do
  echo -e "${GREEN}Introduce la IP del nuevo contenedor:${NC}"
  read -p "> " ip
  if valid_ip "$ip"; then
    ip="$ip/24"
    break
  else
    echo -e "${RED}Formato de IP inválido.${NC}"
  fi
done

while true; do
  echo -e "${GREEN}Introduce la IP del gateway:${NC}"
  read -p "> " gw
  if valid_ip "$gw"; then
    break
  else
    echo -e "${RED}Formato de IP inválido.${NC}"
  fi
done

echo -e "${CYAN}Configurando red...${NC}"
pct set "$new_ct_id" -net0 name=eth0,bridge=vmbr0,ip="$ip",gw="$gw"
```

## (Opcional) Cambiar los parámetros hardware

```bash
clear

while true; do
  echo -e "${GREEN}Introduce el número de cores (ej: 2):${NC}"
  read -p "> " cores
  if [[ "$cores" =~ ^[1-9][0-9]*$ ]]; then
    break
  else
    echo -e "${RED}Número inválido. Debe ser un entero positivo.${NC}"
  fi
done

while true; do
  echo -e "${GREEN}Introduce la memoria RAM en MB (ej: 1024):${NC}"
  read -p "> " memory
  if [[ "$memory" =~ ^[1-9][0-9]*$ ]]; then
    break
  else
    echo -e "${RED}Cantidad inválida. Debe ser un entero positivo.${NC}"
  fi
done

while true; do
  echo -e "${GREEN}Introduce el tamaño de swap en MB (ej: 512):${NC}"
  read -p "> " swap
  if [[ "$swap" =~ ^[0-9]+$ ]]; then
    break
  else
    echo -e "${RED}Cantidad inválida. Debe ser un número entero (puede ser 0).${NC}"
  fi
done

echo -e "${CYAN}Aplicando configuración de hardware...${NC}"
pct set "$new_ct_id" -cores "$cores" -memory "$memory" -swap "$swap"
```

## (Opcional) Ampliar el tamaño del disco

```bash
clear

# Obtener tamaño actual del disco en GB
current_gb=$(pct config "$new_ct_id" | grep -oP 'rootfs:.*?,size=\K[0-9]+(?=G)')

echo -e "${CYAN}Tamaño actual del disco de $new_ct_id: ${NC}${GREEN}${current_gb}G${NC}"

while true; do
  echo -e "${GREEN}Introduce el nuevo tamaño del disco en GB (debe ser mayor que ${current_gb}):${NC}"
  read -p "> " disk_size
  if [[ "$disk_size" =~ ^[1-9][0-9]*$ ]]; then
    if (( disk_size > current_gb )); then
      break
    else
      echo -e "${RED}El nuevo tamaño debe ser mayor que el actual (${current_gb}G).${NC}"
    fi
  else
    echo -e "${RED}Tamaño inválido. Debe ser un número entero positivo.${NC}"
  fi
done

echo -e "${CYAN}Ampliando el disco del contenedor $new_ct_id...${NC}"
pct resize "$new_ct_id" rootfs "${disk_size}G"
```

## Iniciar el contenedor

```bash
pct config "$new_ct_id" 
pct start "$new_ct_id"
```

## Iniciar sesión por SSH mediante claves

Si las claves SSH se han configurado correctamente, se podrá iniciar sesión sin necesidad de contraseña usando el comando:

```bash
ssh manel@<ip_lxc>
```

A partir de aquí ya solo queda "jugar" con este LXC para instalar la aplicación que se necesite ;-)

### Historial de cambios

* **2025-09-10**: Documento inicial
