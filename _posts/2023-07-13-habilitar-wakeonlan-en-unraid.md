---
layout : post
blog-width: true
title: 'Habilitar Wake-On-LAN en Unraid'
date: '2023-07-13 17:43:59'
published: true
tags:
- HomeLab
author:
  display_name: Manel Rodero
---

Este artículo documenta cómo habilitar Wake-On-LAN (WOL) en un servidor con [Unraid 6.11.5](https://unraid.net/) funcionando en un ordenador DIY con una placa base [ASRock B560M Pro4](https://www.asrock.com/MB/Intel/B560M%20Pro4/index.asp).

# Habilitar WOL en la BIOS

* Iniciar el ordenador y acceder a la BIOS usando `F2` o `SUPR`
* Acceder a `Advanced\ACPI Configuration`
* Habilitar **I219 LAN Power On**
* Acceder a `Advanced\Chipset Configuration`
* Cambiar el valor de **Restore on AC/Power Loss** a **Power On**

> **Nota**: Esta última opción se recomienda si se trata de un servidor que necesita volver a ponerse en marcha después de un corte eléctrico.

# Habilitar WOL en Unraid

* Acceder a la interfaz web de Unraid
* Abrir un **Terminal** usando la opción del menú superior derecho
* Ejecutar el comando `ifconfig` para comprobar la dirección IP y la MAC del servidor

```
bond0: flags=5443<UP,BROADCAST,RUNNING,PROMISC,MASTER,MULTICAST>  mtu 1500
        ether aa:bb:cc:dd:ee:ff  txqueuelen 1000  (Ethernet)
        RX packets 506  bytes 130666 (127.6 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 915  bytes 817520 (798.3 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

br0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.80  netmask 255.255.255.0  broadcast 0.0.0.0
        ether aa:bb:cc:dd:ee:ff  txqueuelen 1000  (Ethernet)
        RX packets 477  bytes 118821 (116.0 KiB)
        RX errors 0  dropped 24  overruns 0  frame 0
        TX packets 464  bytes 788749 (770.2 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth0: flags=6211<UP,BROADCAST,RUNNING,SLAVE,MULTICAST>  mtu 1500
        ether aa:bb:cc:dd:ee:ff  txqueuelen 1000  (Ethernet)
        RX packets 506  bytes 130666 (127.6 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 915  bytes 817520 (798.3 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device interrupt 16  memory 0xb3100000-b3120000
```

* Ejecutar el comando `ethtool -s eth0 wol g` para habilitar WOL

> **Nota**: Se utiliza la interfaz física `eth0` y no las interfaces `br0` y `bond0` utilizadas internamente por Unraid.

# Comprobar que funciona

* Apagar el servidor desde `Main` &rarr; `Shutdown`
* Descargar la herramienta `wolcmd.exe` de [Depicus](https://www.depicus.com/wake-on-lan/what-is-wake-on-lan)
* Lanzar una petición de WOL al servidor

```
C:\> wolcmd AA:BB:CC:DD:EE:FF 192.168.1.80 255.255.255.0 7

Wake On Lan signal sent to Mac Address AA:BB:CC:DD:EE:FF
via Broadcast Address 192.168.1.255 on port 7
```

Si todo funciona correctamente, el servidor se debería poner en marcha y responder al comando `ping` en unos instantes:

```
C:\> ping -t 
Pinging 192.168.1.80 with 32 bytes of data:
Reply from 192.168.1.2: Destination host unreachable.
Reply from 192.168.1.2: Destination host unreachable.
Reply from 192.168.1.2: Destination host unreachable.
Reply from 192.168.1.2: Destination host unreachable.
Reply from 192.168.1.2: Destination host unreachable.
Reply from 192.168.1.2: Destination host unreachable.
Reply from 192.168.1.2: Destination host unreachable.
Request timed out.
Reply from 192.168.1.2: Destination host unreachable.
Reply from 192.168.1.2: Destination host unreachable.
Reply from 192.168.1.2: Destination host unreachable.
Reply from 192.168.1.2: Destination host unreachable.
Reply from 192.168.1.2: Destination host unreachable.
Reply from 192.168.1.2: Destination host unreachable.
Reply from 192.168.1.2: Destination host unreachable.
Reply from 192.168.1.2: Destination host unreachable.
Reply from 192.168.1.80: bytes=32 time=1ms TTL=64
Reply from 192.168.1.80: bytes=32 time<1ms TTL=64
Reply from 192.168.1.80: bytes=32 time<1ms TTL=64
[...]
```
