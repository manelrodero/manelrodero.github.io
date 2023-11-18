---
layout : post
blog-width: true
title: '¿Qué memoria tiene mi servidor Proxmox?'
date: '2023-11-18 16:42:22'
published: true
tags:
- Proxmox
author:
  display_name: Manel Rodero
---

Hace unos meses [instalé Proxmox 8.0.2](proxmox-ve-802-en-un-dell-optiplex-7050) en un pequeño [Dell OptiPlex 7050 Micro](mini-pc-dell-optiplex-7050-micro) para usarlo como servidor principal de mi _Home Lab_.

Este equipo, comprado de segunda mano, únicamente tiene **16Gb** de memoria RAM y podría ser interesante ampliarlo a **32Gb** para poder asignar más recursos a los [contenedores Docker en LXC](docker-en-proxmox-lxc-con-turnkey-core) o a las [máquinas virtuales](creacion-de-una-plantilla-de-ubuntu-en-proxmox-ve).

Se, de cuando lo recibí y lo abrí para comprobar que todo estuviese bien, que tiene un único módulo RAM de tipo **SODIMM** (es decir, de tipo portátil) por lo que podría ampliarlo con otro módulo de 16Gb para llegar a los 32Gb.

Para conocer las características del módulo instalado sin tener que abrir de nuevo el equipo, se puede utilizar el comando `dmidecode` tal como se muestra a continuación.

```
root@pve:~# dmidecode --type memory
# dmidecode 3.4
Getting SMBIOS data from sysfs.
SMBIOS 3.0.0 present.

Handle 0x0009, DMI type 16, 23 bytes
Physical Memory Array
        Location: System Board Or Motherboard
        Use: System Memory
        Error Correction Type: None
        Maximum Capacity: 32 GB
        Error Information Handle: Not Provided
        Number Of Devices: 2

Handle 0x000A, DMI type 17, 40 bytes
Memory Device
        Array Handle: 0x0009
        Error Information Handle: Not Provided
        Total Width: 64 bits
        Data Width: 64 bits
        Size: 16 GB
        Form Factor: SODIMM
        Set: None
        Locator: DIMM1
        Bank Locator: Not Specified
        Type: DDR4
        Type Detail: Synchronous Unbuffered (Unregistered)
        Speed: 2400 MT/s
        Manufacturer: 80AD000080AD
        Serial Number: xxxxxxxx
        Asset Tag: 01174700
        Part Number: HMA82GS6AFR8N-UH
        Rank: 2
        Configured Memory Speed: 2400 MT/s
        Minimum Voltage: Unknown
        Maximum Voltage: Unknown
        Configured Voltage: 1.2 V

Handle 0x000B, DMI type 17, 40 bytes
Memory Device
        Array Handle: 0x0009
        Error Information Handle: Not Provided
        Total Width: Unknown
        Data Width: Unknown
        Size: No Module Installed
        Form Factor: Unknown
        Set: None
        Locator: DIMM2
        Bank Locator: Not Specified
        Type: Unknown
        Type Detail: None
```

A partir de la salida del comando `dmidecode` se obtiene la información del módulo instalado en el equipo:

* Tamaño: 16GB
* Formato: SODIMM
* Tipo: DDR4 2400
* Fabricante: 80AD000080AD (Hynix)
* Modelo (P/N): HMA82GS6AFR8N-UH

![HMA82GS6AFR8N-UH][1]

Un módulo idéntico al instalado se puede encontrar en diferentes tiendas _online_ a precios muy diferentes:

* [Memory.net](https://memory.net/product/hma82gs6afr8n-uh-sk-hynix-1x-16gb-ddr4-2400-sodimm-pc4-19200t-s-dual-rank-x8-module/), $44
* [Amazon.es](https://amzn.to/47oJVKL), 99€

Pero también se podría utilizar un módulo compatible **Crucial CT16G4SFD824A**:

* [Crucial.es](https://www.crucial.es/memory/ddr4/ct16g4sfd824a), 61,70€
* [Amazon.es](https://amzn.to/3sH89km), 43€

¡Ahora a esperar al _Black Friday_!

[1]: /assets/img/blog/2023-11-18_image_1.png "HMA82GS6AFR8N-UH"
