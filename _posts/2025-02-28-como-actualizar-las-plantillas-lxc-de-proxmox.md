---
layout : post
blog-width: true
title: '¿Cómo actualizar las plantillas LXC de Proxmox?'
date: '2025-02-28 21:36:11'
#last-updated: '2025-02-28 21:36:11'
published: true
tags:
- Proxmox
author:
  display_name: Manel Rodero
#cover-img: "/assets/img/blog/2025-02-28_cover.png"
thumbnail-img: ""
---

Los **LXC Templates** son plantillas preconfiguradas que se utilizan para crear contenedores LXC en Proxmox.

Las plantillas LXC son imágenes base que ya tienen instalados y configurados ciertos sistemas operativos y aplicaciones, lo que permite desplegar rápidamente nuevos contenedores sin tener que pasar por el proceso de instalación y configuración manual.

# Comandos

## `pveam update`

Este comando sirve para actualizar la **Container Template Database**.

## `pveam list`

Este comando lista todas las plantillas que tenemos instaladas. Se puede indicar un parámetro `<storage>` para listar únicamente las disponibles en ese almacenamiento.

```plaintext
root@pve:~# pveam list local
NAME                                                         SIZE  
local:vztmpl/debian-11-turnkey-core_17.1-1_amd64.tar.gz      197.20MB
local:vztmpl/debian-12-standard_12.2-1_amd64.tar.zst         120.29MB
local:vztmpl/debian-12-standard_12.7-1_amd64.tar.zst         120.65MB
local:vztmpl/debian-12-turnkey-core_18.0-1_amd64.tar.gz      201.70MB
local:vztmpl/debian-12-turnkey-core_18.1-1_amd64.tar.gz      211.29MB
local:vztmpl/ubuntu-22.04-standard_22.04-1_amd64.tar.zst     123.81MB
```

## `pveam available`

Este comando lista todas las plantillas disponibles para descargar. Se puede indicar un parámetro `--section  <mail | system | turnkeylinux>` para limitar el listado a una sección concreta.

```plaintext
root@pve:~# pveam available --section turnkeylinux  | grep -i "core"
turnkeylinux    debian-12-turnkey-asp-net-core_18.0-1_amd64.tar.gz
turnkeylinux    debian-12-turnkey-core_18.1-1_amd64.tar.gz

root@pve:~# pveam available --section system  | grep -i "standard"
system          debian-11-standard_11.7-1_amd64.tar.zst
system          debian-12-standard_12.7-1_amd64.tar.zst
system          devuan-5.0-standard_5.0_amd64.tar.gz
system          ubuntu-20.04-standard_20.04-1_amd64.tar.gz
system          ubuntu-22.04-standard_22.04-1_amd64.tar.zst
system          ubuntu-24.04-standard_24.04-2_amd64.tar.zst
system          ubuntu-24.10-standard_24.10-1_amd64.tar.zst
```

## `pveam download`

Este comando sirve para descargar plantillas de _appliances_. Se debe indicar como parámetros `<storage> <template>` el almacenamiento donde se descargará la plantilla.

```plaintext
root@pve:~# pveam download local ubuntu-24.10-standard_24.10-1_amd64.tar.zst
downloading http://download.proxmox.com/images/system/ubuntu-24.10-standard_24.10-1_amd64.tar.zst to /var/lib/vz/template/cache/ubuntu-24.10-standard_24.10-1_amd64.tar.zst
--2025-02-28 21:46:31--  http://download.proxmox.com/images/system/ubuntu-24.10-standard_24.10-1_amd64.tar.zst
Resolving download.proxmox.com (download.proxmox.com)... 51.91.38.34, 2001:41d0:b00:5900::34
Connecting to download.proxmox.com (download.proxmox.com)|51.91.38.34|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 143472532 (137M) [application/octet-stream]
Saving to: '/var/lib/vz/template/cache/ubuntu-24.10-standard_24.10-1_amd64.tar.zst.tmp_dwnl.2033085'
     0K ........ ........ ........ ........ 23% 10.6M 10s
 32768K ........ ........ ........ ........ 46% 6.83M 9s
 65536K ........ ........ ........ ........ 70% 5.07M 6s
 98304K ........ ........ ........ ........ 93% 4.63M 1s
131072K ........                           100% 8.16M=22s
2025-02-28 21:46:53 (6.22 MB/s) - '/var/lib/vz/template/cache/ubuntu-24.10-standard_24.10-1_amd64.tar.zst.tmp_dwnl.2033085' saved [143472532/143472532]
calculating checksum...OK, checksum verified
download of 'http://download.proxmox.com/images/system/ubuntu-24.10-standard_24.10-1_amd64.tar.zst' to '/var/lib/vz/template/cache/ubuntu-24.10-standard_24.10-1_amd64.tar.zst' finished
```

# `pveam remove`

Este comando sirve para borrar una plantilla. Se debe indicar como parámetro `<template_path>` la localización de la plantilla en nuestro almacenamiento.

```plaintext
pveam remove local:vztmpl/ubuntu-20.04-standard_20.04-1_amd64.tar.gz
```

### Historial de cambios

* **2025-02-28**: Documento inicial
