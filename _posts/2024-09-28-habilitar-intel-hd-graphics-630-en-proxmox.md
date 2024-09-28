---
layout : post
blog-width: true
title: 'Habilitar Intel HD Graphics 630 en Proxmox'
date: '2024-09-28 09:05:27'
#last-updated: '2024-09-26 20:05:27'
published: true
tags:
- Proxmox
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2024-09-28_cover.png"
thumbnail-img: ""
---

Durante la [instalación de **Jellyfin** en Proxmox LXC](instalar-jellyfin-en-proxmox-lxc) he aprovechado para habilitar la **aceleración _hardware_** de la tarjeta [**Intel HD Graphics 630**](https://www.techpowerup.com/gpu-specs/hd-graphics-630.c2962){:target="_blank"}.

# Introducción

Esta tarjeta gráfica está **integrada** en el procesador [Intel i5-7500T](https://ark.intel.com/content/www/us/en/ark/products/97121/intel-core-i5-7500t-processor-6m-cache-up-to-3-30-ghz.html){:target="_blank"} de mi servidor Proxmox, un [Dell OptiPlex 7050 Micro](proxmox-ve-802-en-un-dell-optiplex-7050).

Este procesador, perteneciente a la [**7a generación** de Intel Core i5](https://ark.intel.com/content/www/us/en/ark/products/series/95543/7th-generation-intel-core-i5-processors.html){:target="_blank"} y la familia [**Kaby Lake**](https://ark.intel.com/content/www/us/en/ark/products/codename/82879/products-formerly-kaby-lake.html){:target="_blank"}, tiene las siguientes características:

* 4 nucleos
* velocidad máxima de 3,30GHz
* velocidad base de 2,70GHz
* 6 MB caché
* TDP 35W

La tarjeta gráfica tiene las siguientes características:

* Procesador gráfico Kaby Lake GT2
* Arquitectura de Generación 9.5
* Soporta DirectX 12.1
* Soporta OpenGL 4.6
* Compatible con Intel Quick Sync Video (es la parte que nos interesa)

# Descubrir la GPU en Proxmox

La identificación de una tarjeta gráfica en Proxmox se realiza igual que en cualquier Linux. Se puede utilizar el comando `lspci` para localizarla:

```
root@pve:~# lspci -k | grep -EA3 'VGA|3D|Display'
00:02.0 VGA compatible controller: Intel Corporation HD Graphics 630 (rev 04)
        Subsystem: Dell HD Graphics 630
        Kernel driver in use: i915
        Kernel modules: i915
```

# Drivers de Intel

A continuación, hay que asegurarse que la GPU está disponible como un dispositivo de _render_ **DRI** en Proxmox (p.ej. `/dev/dri/renderD128`). Si no es así, hay que instalar los **_drivers_ de Intel** necesarios para permitir la [aceleración de vídeo mediante _hardware_](https://wiki.debian.org/HardwareVideoAcceleration){:target="_blank"} usando **VA-API**.

Intel divide estos _drivers_ según la **generación** de la tarjeta gráfica y si son o no **gratuitos** tal como se muestra en la siguiente tabla:

| **Licencia \ Generación** | _Gen 8+_ | _Older_ |
|  _Free_ | `intel-media-va-driver` | `i965-va-driver` |
|  _Non-Free_ | `intel-media-va-driver-non-free` | `i965-va-driver-shaders` |

{: .box-note}
Los drivers **_non-free_** son necesarios para **codificar** (_transcoding_) mientras que los gratuitos únicamente decodifican.

## Actualizar repositorios

Para poder instalar los _drivers_ no gratuitos, hay que cambiar los orígenes APT de Proxmox, modificando el fichero `/etc/apt/sources.list`:

```
deb http://ftp.es.debian.org/debian bookworm main contrib non-free non-free-firmware
deb http://ftp.es.debian.org/debian bookworm-updates main contrib non-free non-free-firmware
deb http://security.debian.org bookworm-security main contrib non-free non-free-firmware
```

{: .box-note}
Hay que añadir `non-free` y `non-free-firmware` dado que [Debian dividió los repositorios no gratuitos](https://www.debian.org/releases/bookworm/amd64/release-notes/ch-information.html#non-free-split){:target="_blank"} hace un tiempo.

A continuación se actualiza el sistema:

```
apt update
```

## Instalar `vainfo`

La herramienta [`vainfo`](https://packages.debian.org/vainfo){:target="_blank"} permite comprobar si el soporta de **VA-API** es funcional y con qué _codecs_.

```
apt install vainfo -y
```

## Instalar `intel-gpu-tools`

La herramienta `intel_gpu_top` del paquete [`intel-gpu-tools`](https://packages.debian.org/intel-gpu-tools){:target="_blank"} permite confirmar que se está usando aceleración gráfica:

![Hardware Acceleration][5]

{: .box-note}
Si la sección `Engine/Video` tiene un uso superior al 0%, cuando se reproduce un vídeo, significa que se está usando la aceleración gráfica.

## Instalar _drivers_ no gratuitos

La instalación de los _drivers_ no gratuitos es bastante sencilla después de haber cambiado los repositorios:

```
apt install -y intel-media-va-driver-non-free
# Es recomendable reiniciar el servidor Proxmox después de la instalación
reboot -h now
```

# Comprobar instalación

## VA-API

Al ejecutar `vainfo`, si todo es correcto, se debería mostrar versión de **VA-API** y del **driver de Intel** utilizados, así como los diferentes _profiles_ soportados:

```
root@pve:~# vainfo 
error: can't connect to X server!
libva info: VA-API version 1.17.0
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/iHD_drv_video.so
libva info: Found init function __vaDriverInit_1_17
libva info: va_openDriver() returns 0
vainfo: VA-API version: 1.17 (libva 2.12.0)
vainfo: Driver version: Intel iHD driver for Intel(R) Gen Graphics - 23.1.1 ()
vainfo: Supported profile and entrypoints
      VAProfileNone                   : VAEntrypointVideoProc
      VAProfileNone                   : VAEntrypointStats
      VAProfileMPEG2Simple            : VAEntrypointVLD
      VAProfileMPEG2Simple            : VAEntrypointEncSlice
      VAProfileMPEG2Main              : VAEntrypointVLD
[...]
      VAProfileHEVCMain10             : VAEntrypointVLD
      VAProfileHEVCMain10             : VAEntrypointEncSlice
      VAProfileVP9Profile0            : VAEntrypointVLD
      VAProfileVP9Profile2            : VAEntrypointVLD
```

## Card / Render

Se lista el contenido del directorio `/dev/dri`:

```
root@pve:~#  ls -lh /dev/dri
total 0
drwxr-xr-x 2 root root         80 Sep 26 20:53 by-path
crw-rw---- 1 root video  226,   1 Sep 26 20:53 card1
crw-rw---- 1 root render 226, 128 Sep 26 20:53 renderD128
```

En mi caso, hay dos dispositivos:

* `/dev/dri/card1` (ID 226,1) que representa la tarjeta gráfica
* `/dev/dri/renderD128` (ID 226,128) que es usado para la renderización de gráficos

# Referencias

* [Setting up Intel GPU passthrough on Proxmox LXC containers](https://geekistheway.com/2022/12/23/setting-up-intel-gpu-passthrough-on-proxmox-lxc-containers/){:target="_blank"}
* [How to Install Plex on Proxmox](https://www.wundertech.net/how-to-install-plex-on-proxmox/){:target="_blank"} + Hardware Acceleration
* [Intel Quick Sync Video](https://en.wikipedia.org/wiki/Intel_Quick_Sync_Video){:target="_blank"}
* [Intel Graphics Media Driver](https://github.com/intel/media-driver) para soportar codificación/decodificación/procesamiento de vídeo mediante _hardware_
* [Intel media stack on Ubuntu](https://github.com/Intel-Media-SDK/MediaSDK/wiki/Intel-media-stack-on-Ubuntu){:target="_blank"}
* [HWA Tutorial On Intel GPU](https://jellyfin.org/docs/general/administration/hardware-acceleration/intel/){:target="_blank"} by Jellyfin
* [Using an Intel GPU for Jellyfin Hardware Acceleration](https://blog.stabl.one/posts/007-jellyfin-hardware-acceleration/)

### Historial de cambios

* **2024-09-28**: Documento inicial

[1]: /assets/img/blog/2024-09-28_image_5.png "Hardware Acceleration"
