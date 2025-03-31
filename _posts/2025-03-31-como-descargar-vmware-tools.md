---
layout : post
blog-width: true
title: '¿Cómo descargar VMware Tools?'
date: '2025-03-31 10:37:29'
#last-updated: '2025-03-31 10:37:29'
published: true
tags:
- VMware
author:
  display_name: Manel Rodero
#cover-img: "/assets/img/blog/2025-03-31_cover.png"
thumbnail-img: ""
---

Aprovechando la actualización de las **VMware Tools** a la versión [**12.5.1**](https://techdocs.broadcom.com/us/en/vmware-cis/vsphere/tools/12-5-0/release-notes/vmware-tools-1251-release-notes.html){:target="_blank"} debido al [CVE-2025-22230](https://support.broadcom.com/web/ecx/support-content-notification/-/external/content/SecurityAdvisories/0/25518){:target="_blank"} del 25 de marzo de 2025, explicáre cómo descargar estas herramientas desde la web de Broadcom.

# VMware Tools

Las VMware Tools son un conjunto de utilidades y controladores diseñados para mejorar la funcionalidad de las máquinas virtuales que se ejecutan en VMware Workstation, VMware Player o VMware ESXi.

Estas herramientas permiten una integración más fluida entre el sistema operativo invitado (el que se ejecuta en la máquina virtual) y el anfitrión (el sistema operativo principal).

Algunas de sus principales funciones incluyen:

- **Mejor rendimiento:** Optimizan el rendimiento gráfico y la interacción del ratón.
- **Sincronización:** Facilitan la sincronización de tiempo entre el anfitrión y la máquina virtual.
- **Funciones avanzadas:** Permiten copiar y pegar texto, arrastrar y soltar archivos entre el anfitrión y la máquina virtual.
- **Gestión:** Habilitan operaciones como el apagado o reinicio desde la interfaz del software VMware.
- **Redimensionado dinámico:** Ajustan automáticamente la resolución de pantalla de la máquina virtual al tamaño de la ventana.

## Descarga

Para descargar las VMware Tools hay que hacer lo siguiente:

- Acceder a la [página de descarga de las VMware Tools](https://support.broadcom.com/group/ecx/productdownloads?subfamily=VMware%20Tools&freeDownloads=true){:target="_blank"} iniciando sesión en el portal de soporte de Broadcom
- Sleccionar la versión **VMware Tools 12.x** y, a continuación, seleccionar la _release_ **12.5.1**
- Aparecerá una lista de paquetes de instalación y otros archivos disponibles:
  - **VMware Tools packages for Windows** (`VMware-Tools-windows-12.5.1-24649672.zip`)
  - VMware Tools packages for Windows Arm
  - VMware Tools for Windows Arm, in-guest installer
  - VMware Tools packages for GuestStore
  - **VMware Tools for Windows, 64-bit in-guest installer** (`VMware-tools-12.5.1-24649672-x64.exe.zip`)
  - **VMware Tools Offline VIB Bundle** (`VMware-Tools-12.5.1-core-offline-depot-ESXi-all-24649672.zip`)
  - VMware Tools packages for Windows
  - VMware Tools for Windows, 32-bit in-guest installer
- Hacer clic en el icono de descarga de los tres paquetes marcados en **negrita** en la lista anterior que contienen:
  - Ficheros _Floppy_ y `windows.iso`
  - Instalador para sistemas operativos Windows x64
  - _Bundle_ para actualizar servidores ESXi

![VMware Tools][1]

{: .box-note}
Para que aparezca el icono de descarga al lado de los paquetes es necesario aceptar los [términos y condiciones](https://www.broadcom.com/company/legal/licensing){:target="_blank"} de Broadcom al inicio de la página.

# Referencias

- [Downloading VMware Tools](https://knowledge.broadcom.com/external/article/368758){:target="_blank"}
- [Build numbers and versions of VMware Tools](https://knowledge.broadcom.com/external/article/304809){:target="_blank"}
- [VMware Tools compatibility with guest operating systems](https://knowledge.broadcom.com/external/article/313371){:target="_blank"}
- [Install VMware Tools in VMware products](https://knowledge.broadcom.com/external/article/315363){:target="_blank"}
- [Installing and upgrading VMware Tools in vSphere](https://knowledge.broadcom.com/external/article/316546){:target="_blank"}
- [Index of tools/releases/12.5.1/windows/](https://packages.vmware.com/tools/releases/12.5.1/windows/){:target="_blank"} (**acceso abierto de momento**)
- [VMware Tools 12.5.0 is ready for the latest Windows operating systems](https://blogs.vmware.com/cloud-foundation/2024/11/07/vmware-tools-12-5-0-is-ready-for-the-latest-windows-operating-systems/){:target="_blank"}

### Historial de cambios

- **2025-03-31**: Documento inicial

[1]: /assets/img/blog/2025-03-31_image_1.png "VMware Tools"
