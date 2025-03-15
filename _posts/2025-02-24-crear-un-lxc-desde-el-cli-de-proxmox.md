---
layout : post
blog-width: true
title: 'Crear un LXC desde el CLI de Proxmox'
date: '2025-02-24 19:48:30'
#last-updated: '2025-02-24 19:48:30'
published: true
tags:
- Proxmox
author:
  display_name: Manel Rodero
#cover-img: "/assets/img/blog/2025-02-24_cover.png"
thumbnail-img: ""
---

Cuando comienzas a utilizar Proxmox, sueles utilizar la GUI para crear los LXC pero llega un momento en que necesitas algo más de rapidez que hacerlo mediante este asistente.

El comando que suelo utilizar para crear los contenedores LXC es el siguiente:

```bash
pct create $ct_id local:vztmpl/debian-12-turnkey-core_18.0-1_amd64.tar.gz \
  --ostype debian --arch amd64 \
  --hostname $hostname --unprivileged $unprivileged --password $password --ssh-public-keys /root/id_edcsa.pub \
  --storage local-lvm --rootfs volume=local-lvm:${disk_size} \
  --cores $cores \
  --memory $memory --swap $default_swap \
  --net0 name=eth0,bridge=vmbr0,firewall=1,ip=${ip}/24,gw=$default_gateway \
  --nameserver "$nameserver" \
  --start false
```

La explicación de cada parámetro es la siguiente:

* `$ct_id`: Identificador del contenedor que se creará
* `local:vztmpl/debian-12-turnkey-core_18.0-1_amd64.tar.gz`: Usar la plantilla de Debian 12 TurnKey Core de la ubicación `local`
* `--ostype debian`: Especifica el tipo de sistema operativo como Debian
* `--arch amd64`: Arquitectura del contenedor, en este caso AMD64
* `--hostname $hostname`: Nombre de host del contenedor
* `--unprivileged $unprivileged`: Indica si el contenedor debe ser no privilegiado (_unprivileged_)
* `--password $password`: Contraseña para el usuario `root` del contenedor
* `--ssh-public-keys /root/id_edcsa.pub`: Llave pública SSH para acceder al contenedor
* `--storage local-lvm`: Almacenamiento del contenedor en `local-lvm`
* `--rootfs volume=local-lvm:${disk_size}`: Sistema de archivos raíz del contenedor con un tamaño específico
* `--cores $cores`: Número de núcleos de CPU asignados al contenedor
* `--memory $memory`: Cantidad de memoria asignada al contenedor
* `--swap $default_swap`: Cantidad de memoria swap asignada al contenedor
* `--net0`: Configuración de red para la interfaz `eth0`
  * `bridge=vmbr0`: Usa el bridge `vmbr0` para la red
  * `firewall=1`: Habilita el firewall para esta interfaz
  * `ip=${ip}/24`: Dirección IP del contenedor con una máscara de subred /24
  * `gw=$default_gateway`: Puerta de enlace predeterminada para el contenedor
* `--nameserver "$nameserver"`: Especifica el servidor DNS para el contenedor
* `--start false`: Indica que el contenedor no debe arrancar automáticamente después de ser creado

Como se puede observar, se utilizan variables en muchos valores porque este comando se integra en un pequeño _shell script_ [`create_ct.sh`](https://gist.github.com/manelrodero/ab36b56690a8f7b565cdd5c22fd33476){:target="_blank"} para preguntarlos.

Si no se especifica lo contrario, los valores predeterminados para mi entorno los siguientes:

```plaintext
# Valores predeterminados
default_privileged="no"
default_docker="no"
default_disk_size=8
default_memory=512
default_cores=1
default_nameserver="192.168.1.81 192.168.1.82"
default_mount="no"
default_swap=512
default_gateway="192.168.1.1"
```

# Ejecución

A continuación se detalla un ejemplo de ejecución del _script_:

```plaintext
root@pve:~# ./create_ct.sh 
Introduce el CT ID: 99
Introduce el nombre del contenedor: test
Privilegiado (yes/no) [no]: 
¿Utilizarás Docker? (yes/no) [no]: 
Introduce el password del usuario 'root': 
Confirma el password del usuario 'root': 
Introduce el tamaño del disco (GB) [8]: 4
Introduce la memoria RAM (MB) [512]: 
Introduce el número de cores [1]: 
Introduce la dirección IP (192.168.1.x): 192.168.1.99
Introduce los servidores DNS (separados por espacios) [192.168.1.81 192.168.1.82]: 
¿Quieres montar /datos/samba/media en el CT? (yes/no) [no]: 
Resumen de datos:
CT ID: 99
Nombre del contenedor: test
Privilegiado: no
Utilizará Docker: no
Password: [oculto]
Tamaño del disco: 4GB
Memoria RAM: 512MB
Cores: 1
IP: 192.168.1.99
Servidores DNS: 192.168.1.81 192.168.1.82
Montar /datos/samba/media: no
Memoria SWAP: 512MB
Gateway: 192.168.1.1
¿Son correctos estos datos? (yes/no) [no]: yes
  Logical volume "vm-99-disk-0" created.
Creating filesystem with 1048576 4k blocks and 262144 inodes
Filesystem UUID: c03261bb-dc48-4126-a4d9-e6c67662830e
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736
extracting archive '/var/lib/vz/template/cache/debian-12-turnkey-core_18.0-1_amd64.tar.gz'
Total bytes read: 545402880 (521MiB, 110MiB/s)
Creating SSH host key 'ssh_host_ecdsa_key' - this may take some time ...
done: SHA256:<redacted> root@test
Creating SSH host key 'ssh_host_ed25519_key' - this may take some time ...
done: SHA256:<redacted> root@test
Creating SSH host key 'ssh_host_dsa_key' - this may take some time ...
done: SHA256:<redacted> root@test
Creating SSH host key 'ssh_host_rsa_key' - this may take some time ...
done: SHA256:<redacted> root@test
```

# Referencias

* [pct - Tool to manage Linux Containers (LXC) on Proxmox VE](https://pve.proxmox.com/pve-docs/pct.1.html){:target="_blank"}
* [Proxmox VE Helper-Scripts: Debian LXC](https://community-scripts.github.io/ProxmoxVE/scripts?id=debian){:target="_blank"}

### Historial de cambios

* **2025-02-24**: Documento inicial
