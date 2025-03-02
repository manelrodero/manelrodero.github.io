---
layout : post
blog-width: true
title: 'Descarga de Veeam Backup & Replication Community Edition'
date: '2021-07-02 09:37:50'
published: true
tags:
- Software
author:
  display_name: Manel Rodero
---

[Veeam Backup & Replication Community Edition](https://www.veeam.com/virtual-machine-backup-solution-free.html){:target="_blank"} es una versión gratuita que permite proteger hasta 10 cargas de trabajo (VMware, Hyper-V, NAS, etc.).

Para descargar el producto es necesario [registrarse con un correo electrónico profesional](https://www.veeam.com/signin.html){:target="_blank"}. Esta cuenta registrada permite descargar la _Community Edition_, probar las versiones comerciales de los productos de Veeam con todas sus funcionalidades, acceder a recursos educativos gratuitos, etc.

Los pasos para descargar son los siguientes:

- Acceder a la página [Veeam Backup & Replication Community Edition](https://www.veeam.com/virtual-machine-backup-solution-free.html){:target="_blank"}
- Seleccionar el botón [`Download Free`](https://www.veeam.com/virtual-machine-backup-solution-free-download.html){:target="_blank"}
- Iniciar sesión mediante la cuenta creada anteriormente
- Descargar el producto (a 2 de Julio de 2021, la versión **11.0.0.837**):
  - [VeeamBackup&Replication_11.0.0.837_20210525.iso](https://download2.veeam.com/VBR/v11/VeeamBackup&Replication_11.0.0.837_20210525.iso){:target="_blank"}
  - [Release Notes 11.0](https://www.veeam.com/veeam_backup_11_0_release_notes_en_rn.pdf){:target="_blank"}

Una vez descargado se comprueba el _hash_ `SHA1` del fichero ISO usando PowerShell:

```PowerShell
(Get-FileHash 'VeeamBackup&Replication_11.0.0.837_20210525.iso' -Algorithm SHA1).Hash -eq '1b2fa3bff775049670a5f02d927b0f77262deb90'
```

## Documentación

En un correo electrónico que llega después de haber descargado el fichero se indica que **no es necesario tener una _license key_ para usar la versión gratuita** y se proporcionan enlaces a la documentación del producto:

- [User Guide for VMware vSphere](https://www.veeam.com/veeam_backup_11_0_user_guide_vsphere_pg.pdf){:target="_blank"}
- [User Guide for Microsoft Hyper-V](https://www.veeam.com/veeam_backup_11_0_user_guide_hyperv_pg.pdf){:target="_blank"}
- [Quick Start Guide for VMware vSphere](https://www.veeam.com/veeam_backup_11_0_quick_start_guide_vsphere_pg.pdf){:target="_blank"}
- [Quick Start Guide for Microsoft Hyper-V](https://www.veeam.com/veeam_backup_11_0_quick_start_guide_hyperv_pg.pdf){:target="_blank"}

También se indica la [página para descargar actualizaciones del producto](https://www.veeam.com/download-version.html){:target="_blank"} así como la página de acceso a la información personal [My Veeam](https://my.veeam.com/):

![My Veeam][1]

[1]: /assets/img/blog/2021-07-02_image_1.png "My Veeam"
