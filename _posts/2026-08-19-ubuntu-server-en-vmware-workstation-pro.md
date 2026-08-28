---
layout : post
blog-width: true
title: 'Ubuntu Server en VMware Workstation Pro'
date: '2026-08-19 11:07:42'
last-updated: '2026-08-28 13:22:42'
published: true
tags:
- Software
author:
  display_name: Manel Rodero
#cover-img: "/assets/img/blog/2026-08-19_cover.png"
thumbnail-img: ""
---

Este artículo recoge la configuración que suelo utilizar a la hora de crear máquinas virtuales de **Ubuntu Server 26.04** en **VMware Workstation Pro 26H1**.

Se instalará y configurará Ubuntu Server explicando cada ajuste técnico y los motivos por los que es aconsejable aplicarlo. El objetivo es obtener un entorno estable, rápido y adecuado para desarrollo, pruebas y laboratorio personal.

## 1. Configuración de CPU

### 1.1 Procesador y núcleos

Se recomienda configurar **1 procesador** con **2 núcleos**.

Linux gestiona mejor un único socket con varios núcleos que múltiples sockets con un solo núcleo cada uno. Esto reduce la latencia, evita la sobrecarga NUMA y mejora el rendimiento general del sistema.

### 1.2 Virtualize Intel VT‑x/EPT

Normalmente es **muy recomendable activar esta opción**, ya que permite:

- aceleración por hardware
- mejor rendimiento en Docker, LXC, KVM y QEMU
- menor latencia en operaciones de virtualización
- mayor velocidad en compilaciones y criptografía

Sin embargo, en este equipo **no se activará** porque Windows 11 ejecuta un hipervisor interno debido a características del sistema como WSL2 y la seguridad basada en virtualización (VBS).

Cuando el hipervisor de Windows está activo, VMware Workstation Pro **no puede acceder a VT‑x/EPT**, por lo que esta opción queda inutilizable. La máquina virtual funcionará correctamente, pero sin aceleración por hardware.

### 1.3 Virtualize IOMMU

Es recomendable **activarlo**.

IOMMU mejora la gestión de memoria, el rendimiento de contenedores y la seguridad frente a accesos DMA. También aporta beneficios en entornos con Kubernetes o cargas de trabajo con aislamiento avanzado.

### 1.4 Virtualize CPU performance counters

Debe dejarse **desactivado**.

Estos contadores solo son útiles para análisis de rendimiento avanzados (profiling) y añaden sobrecarga innecesaria en un servidor.

## 2. Memoria RAM

Asignar **4 GB** (4096MB) es suficiente para la mayoría de servicios.
Si se van a ejecutar contenedores, bases de datos o servicios más pesados, se puede aumentar a **8 GB** sin problema.

## 3. Almacenamiento

### 3.1 Tipo de disco: NVMe

VMware Workstation Pro permite utilizar discos NVMe, que ofrecen:

- menor latencia
- mayor número de IOPS
- mejor rendimiento en bases de datos y contenedores
- detección nativa en Ubuntu (`/dev/nvme0n1`)

### 3.2 Controladora: Paravirtualized SCSI (PVSCSI)

Es la controladora más eficiente para entornos virtualizados:

- menor consumo de CPU
- mayor rendimiento en operaciones de E/S
- optimizada para cargas de trabajo intensivas
- totalmente compatible con Ubuntu Server

### 3.3 Tamaño del disco

Se recomienda asignar **50 GB**, suficiente para un servidor base con Docker y servicios adicionales.

### 3.4 Formato del disco

- **Thin provisioned**
- **Single file**

## 4. Firmware

### 4.1 UEFI

Ubuntu Server funciona mejor con UEFI que con BIOS:

- arranque más rápido
- compatibilidad moderna
- mejor integración con NVMe

## 5. Red

### 5.1 NAT

Es la opción más estable en VMware Workstation Pro:

- salida a Internet sin configuración adicional
- IP consistente
- menos problemas con firewalls o routers
- ideal para entornos de desarrollo

## 6. Guest Isolation

### 6.1 Copy & Paste
Debe dejarse **activado**.

### 6.2 Drag & Drop
Debe dejarse **desactivado**.

## 7. Dispositivos virtuales

### 7.1 USB Controller
Puede dejarse activado.

### 7.2 Tarjeta de sonido
Debe eliminarse.

### 7.3 Cámara e impresora
No deben añadirse.

## 8. Mitigaciones de side‑channel

Estas mitigaciones reducen el rendimiento y no aportan beneficios reales en un entorno personal o de laboratorio.
Es recomendable **desactivarlas**.

## 9. Instalación del sistema operativo

### 9.1 Instalación de Ubuntu Server

Una vez configurada la máquina virtual:

1. Arrancar la VM.
2. Instalar Ubuntu Server normalmente (instalar el servidor SSH)
3. Actualizar el sistema:

```bash
sudo apt update
sudo apt full-upgrade -y
sudo apt autoremove -y
sudo apt clean
sudo timedatectl set-ntp true
sudo reboot
```

### 9.2 Instalación de PowerShell

La instalación de [PowerShell 7](https://learn.microsoft.com/en-us/powershell/scripting/install/install-ubuntu?view=powershell-7.6) desde el repositorio de paquetes de Microsoft aún no es posible porque no está publicada la versión para Ubuntu 26.04:

```bash
#!/bin/bash
###################################
# Prerequisites

# Update the list of packages
sudo apt update

# Install pre-requisite packages.
sudo apt install -y wget apt-transport-https software-properties-common

# Get the version of Ubuntu
source /etc/os-release

# Download the Microsoft repository keys
wget -q https://packages.microsoft.com/config/ubuntu/$VERSION_ID/packages-microsoft-prod.deb

# Register the Microsoft repository keys
sudo dpkg -i packages-microsoft-prod.deb

# Delete the Microsoft repository keys file
rm packages-microsoft-prod.deb

# Update the list of packages after we added packages.microsoft.com
sudo apt update

###################################
# Install PowerShell
sudo apt install -y powershell
```

Por tanto, hay que realizar la instalación manual de la versión deseada (en este caso la **7.6.5**):

```bash
#!/bin/bash
###################################
# Prerequisites

# Update the list of packages
sudo apt update

# Install pre-requisite packages.
sudo apt install -y wget

# Download the PowerShell package file
wget https://github.com/PowerShell/PowerShell/releases/download/v7.6.5/powershell_7.6.5-1.deb_amd64.deb

###################################
# Install the PowerShell package
sudo dpkg -i powershell_7.6.5-1.deb_amd64.deb

# Resolve missing dependencies and finish the install (if necessary)
sudo apt install -f

# Delete the downloaded package file
rm powershell_7.6.5-1.deb_amd64.deb
```

### 9.3 Ejecutar PowerShell

Para comprobar que PowerShell está instalado:

```bash
pwsh
```

Para obtener la versión:

```bash
pwsh -v
```

Para actualizar:

```bash
sudo apt update
sudo apt install --only-upgrade powershell
```

## 10. Instalación de VMware Tools

> **Nota**: En principio, deberían haberse instalado durante el proceso de instalación del SO.

```bash
sudo apt install open-vm-tools -y
```

## 11. Instalación de Docker antes del snapshot

Es aconsejable instalar Docker antes de crear el snapshot base, ya que permite reutilizar la máquina virtual como plantilla para futuros entornos de desarrollo y laboratorio.

### Instalación oficial de Docker (formato moderno con docker.sources)

#### 1️⃣ Preparar el sistema

```bash
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

#### 2️⃣ Añadir el repositorio oficial

```bash
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

#### 3️⃣ Actualizar repositorios

```bash
sudo apt update
```

#### 4️⃣ Instalar Docker Engine + CLI + containerd

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

#### 5️⃣ Añadir el usuario al grupo docker

```bash
sudo usermod -aG docker $USER
```

#### 6️⃣ Reiniciar sesión

#### 7️⃣ Verificar Docker

```bash
docker --version
docker info
docker run hello-world
```

## 12. Mostrar la IP asignada en la consola de Ubuntu Server

Para mostrar la dirección IP directamente en la consola de VMware (antes del login), es necesario modificar el archivo `/etc/issue`, que controla el mensaje mostrado por el servicio getty.

Editar el archivo:

```bash
sudo nano /etc/issue
```

Añadir al final (ajustando la interfaz de red según el sistema):

```bash
IP principal: \4{ens33}
```

Reiniciar el servicio getty:

```bash
sudo systemctl restart getty@tty1.service
```

A partir de este momento, la consola mostrará la IP asignada por DHCP justo antes del prompt de login.

## 13. Snapshot final

Apagar la máquina:

```bash
sudo poweroff
```

Crear snapshot limpio:

**Base Updated + PowerShell + Docker**

## 14. Configuración de la VM clones y laboratorios

Cuando se clona una máquina virtual a partir del snapshot base, es recomendable ajustar algunos parámetros fundamentales:

- el **nombre del host**, para identificar cada instancia (ubuntu1, ubuntu2, wazuh-manager, wazuh-agent1, etc.)
- la **dirección IP**, para evitar conflictos y permitir la creación de laboratorios (Wazuh, Elastic, Docker Swarm, etc.)
- las **claves SSH**, para evitar duplicados de los hosts conocidos y las advertencias del cliente SSH

## 14.1 Cambio del nombre del host

Ubuntu Server utiliza dos mecanismos:

- `/etc/hostname` → nombre persistente
- `/etc/hosts` → resolución local
- `cloud-init` → puede sobrescribir el hostname si no se desactiva

### 1️⃣ Cambiar el hostname

```bash
sudo hostnamectl set-hostname ubuntu1
```

### 2️⃣ Actualizar /etc/hosts

Editar:

```bash
sudo nano /etc/hosts
```

Modificar la línea:

```plaintext
127.0.1.1   ubuntu1
```

### 3️⃣ Desactivar cloud-init para que no sobrescriba el hostname

Crear archivo:

```bash
sudo nano /etc/cloud/cloud.cfg.d/99-disable-hostname.cfg
```

Contenido:

```plaintext
preserve_hostname: true
```

### 4️⃣ Reiniciar

```bash
sudo reboot
```

Tras el reinicio, el sistema mostrará el nuevo nombre del host en la consola y por SSH.

## 14.2 Configuración de IP estática (Netplan)

Ubuntu Server 26.04 utiliza **Netplan** para gestionar la red.

Para asignar una IP estática en VMware Workstation (NAT o Bridged), se debe editar el archivo de configuración correspondiente.

### 1️⃣ Identificar la interfaz de red

Normalmente en VMware es:

- `ens33`
- `eth0`

Comprobar:

```bash
ip -4 a
```

### 2️⃣ Editar la configuración de Netplan

```bash
sudo nano /etc/netplan/01-netcfg.yaml
```

### 3️⃣ Configuración de ejemplo (NAT en VMware)

```plaintext
network:
  version: 2
  renderer: networkd
  ethernets:
    ens33:
      dhcp4: false
      addresses:
        - 192.168.20.11/24
      routes:
        - to: default
          via: 192.168.20.2
      nameservers:
        addresses:
          - 1.1.1.1
          - 8.8.8.8
```

```bash
sudo chmod 600 /etc/netplan/01-netcfg.yaml
sudo mv /etc/netplan/00-installer-config.yaml /etc/netplan/00-installer-config.yaml.bak
```

### 4️⃣ Aplicar la configuración

> **Nota**: A la hora de aplicar la configuración, es mejor hacerlo desde la consola para evitar problemas con la conexión SSH.

```bash
sudo netplan apply
```

### 5️⃣ Verificar

```bash
ip a
ping -c 3 192.168.20.2
ping -c 3 google.com
```

## 14.3 Regenerar las claves SSH

Se ejecutarán los dos scripts existentes al final del artículo [Autenticación SSH usando una clave privada](https://www.manelrodero.com/blog/autenticacion-ssh-usando-una-clave-privada) para **generar nuevas claves SSH** y **configurar el servidor SSH para usarlas**.

La ejecución del primer script `sudo ./ssh1.sh` elimina antiguas claves DSA y EDCSA y genera claves **ED25519** y **RSA (3072 bits)**:

```bash
#!/bin/bash

set -e

echo "🔐 Eliminando claves antiguas DSA y ECDSA si existen..."
rm -f /etc/ssh/ssh_host_dsa_key*
rm -f /etc/ssh/ssh_host_ecdsa_key*

echo "🔑 Regenerando clave ED25519..."
yes | ssh-keygen -t ed25519 -f /etc/ssh/ssh_host_ed25519_key -N ""

echo "🔑 Regenerando clave RSA (3072 bits)..."
yes | ssh-keygen -t rsa -b 3072 -f /etc/ssh/ssh_host_rsa_key -N ""

echo "✅ Claves SSH regeneradas correctamente."
```

La ejecución del segundo script `sudo ./ssh2.sh` configura el servidor de SSH para utilizar únicamente las nuevas claves generadas anteriormente:

```bash
#!/bin/bash

CONFIG="/etc/ssh/sshd_config"
BACKUP="/etc/ssh/sshd_config.bak"

# Crear copia de seguridad
cp "$CONFIG" "$BACKUP"

# Procesar el archivo
awk '
BEGIN { found_pubkey = 0 }
{
    if ($0 ~ /^[# ]*HostKey[ \t]+\/etc\/ssh\/ssh_host_rsa_key/) {
        print "HostKey /etc/ssh/ssh_host_rsa_key"
    } else if ($0 ~ /^[# ]*HostKey[ \t]+\/etc\/ssh\/ssh_host_ed25519_key/) {
        print "HostKey /etc/ssh/ssh_host_ed25519_key"
    } else if ($0 ~ /^[^#]*HostKey[ \t]+\/etc\/ssh\/ssh_host_dsa_key/) {
        print "#" $0
    } else if ($0 ~ /^[^#]*HostKey[ \t]+\/etc\/ssh\/ssh_host_ecdsa_key/) {
        print "#" $0
    } else if ($0 ~ /^PubkeyAcceptedKeyTypes/) {
        print "PubkeyAcceptedKeyTypes ssh-ed25519,ssh-rsa"
        found_pubkey = 1
    } else {
        print $0
    }
}
END {
    if (found_pubkey == 0) {
        print ""
        print "PubkeyAcceptedKeyTypes ssh-ed25519,ssh-rsa"
    }
}
' "$BACKUP" > "$CONFIG"

echo "✅ Archivo actualizado: $CONFIG"
```

Una vez configurado el servidor SSH, lo mejor es reiniciar el servicio o el servidor para aplicar los cambios.

### Historial de cambios

* **2026-08-19**: Documento inicial
* **2026-08-28**: Revisión y mejoras
