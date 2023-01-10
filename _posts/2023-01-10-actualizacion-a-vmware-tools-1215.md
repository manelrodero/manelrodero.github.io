---
layout : post
blog-width: true
title: 'Actualización a VMware Tools 12.1.5'
date: '2023-01-10 18:16:18'
published: true
tags:
- Software
author:
  display_name: Manel Rodero
---

# Estado actual

El [VMSA-2022-0029](https://www.vmware.com/content/vmware/vmware-published-sites/us/security/advisories/VMSA-2022-0029.html) del pasado 29 de Noviembre de 2022 avisa de una vulnerabilidad en **VMware Tools for Windows**.

En el servidor que tenemos que actualizar, un **Dell Inc. PowerEdge R340**, está instalada la versión **12.1.0 (20219665)** de estas herramientas.

Desde la versión 6.7U2 de VMware ESXi las [VMware Tools no se actualizan con el servidor](https://blogs.vmware.com/vsphere/2019/04/vsphere-6-7-u2-vmware-tools-compatibility.html) y es necesario actualizarlas usando [Update Manager](https://uncomplicatingit.com/vmware/adding-vmware-tools-11-0-5-to-esxi-hosts/), [Lifecycle Manager](https://thesleepyadmins.com/2021/07/01/updating-vmware-tools-on-esxi-7-0-host-using-vmware-lifecycle-manager/) o [de forma manual](https://www.vladan.fr/vmware-tools-offline-vib-for-esxi-host-bundle-download-and-install/).

# 12.1.5

La descarga de las [**VMware Tools**](https://www.vmware.com/go/tools) para actualizar la versión existente en el servidor ESXi se realiza de la siguiente manera:

* Acceder al _dashboard_ de [VMware Customer Connect](https://customerconnect.vmware.com/dashboard)
* Iniciar sesión con nuestras credenciales
* Seleccionar `Products` > `All Products`
* Localizar el producto **VMware Tools**
* Hacer click en `View Download Components`
* Seleccionar `12.X` en el desplegable de versiones
* Buscar la versión actual (en este caso la [**12.1.5**](https://customerconnect.vmware.com/downloads/details?downloadGroup=VMTOOLS1215&productId=1259&rPId=97424))
* Hacer click en `Go to downloads`

Desde aquí se pueden descargar diferentes versiones de las herramientas:

* VMware Tools packages for Windows (`*.zip`)
* VMware Tools packages for Windows (`*.gz`)
* VMware Tools for Windows, 32-bit in-guest installer (`*.zip`)
* VMware Tools for Windows, 64-bit in-guest installer (`*.zip`)
* VMware Tools packages for GuestStore (`*.zip`)
* VMware Tools packages for GuestStore (`*.gz`)
* VMware Tools Offline VIB Bundle (`*.zip`)

En este caso se descarga el _offline bundle_ (`VMware-Tools-12.1.5-core-offline-depot-ESXi-all-20735119.zip`) para actualizar el servidor ESXi siguiendo estos pasos:

* Subir el fichero `*.zip` a un _datastore_ del servidor ESXi
* Parar las máquinas virtuales que se estén ejecutando en el servidor
* `Host` > `Actions` > `Enter maintenance mode`

> **Nota**: También se podría ejecutar el comando `esxcli system maintenanceMode set -e true`.

> **Nota**: Si hay algún error "_Another task is already in progress._" se ejecutaría el comando `/etc/init.d/hostd restart && /etc/init.d/vpxa restart`.

* `Host` > `Actions` > `Services` > `Enable Secure Shell (SSH)`
* Conectarse al servidor mediante un cliente SSH
* Comprobar la versión nstalada actualmente mediante el comando `esxcli software vib list`:

```
[root@esxi:~] esxcli software vib list | grep tools
tools-light                    12.1.0.20219665-20295239               VMware  VMwareCertified   2022-09-02
```

* Actualizar mediante el comando `esxcli software vib update`:

```
[root@esxi:~] esxcli software vib update --depot=/vmfs/volumes/datastore1/Updates/VMware-Tools-12.1.5-core
-offline-depot-ESXi-all-20735119.zip
Installation Result
   Message: Operation finished successfully.
   Reboot Required: false
   VIBs Installed: VMware_locker_tools-light_12.1.5.20735119-20735876
   VIBs Removed: VMware_locker_tools-light_12.1.0.20219665-20295239
   VIBs Skipped:
```

* Comprobar que se han actualizado correctamente:

```
[root@esxi:~] esxcli software vib list | grep tools
tools-light                    12.1.5.20735119-20735876               VMware  VMwareCertified   2023-01-10
```

* `Host` > `Actions` > `Services` > `Disable Secure Shell (SSH)`
* `Host` > `Actions` > `Exit maintenance mode`

> **Nota**: También se podría ejecutar el comando `esxcli system maintenanceMode set -e false`.

* Poner en marcha las VMs del servidor

# Actualización de las VMs

La actualización de las VMware Tools en las máquinas virtuales se puede realizar de diferentes maneras:

* Actualización automática cuando se inicia la VM
* Actualización manual a una o más máquinas virtuales a través de la interfaz de usuario de vSphere
* VMware Update Manager (inmediato, programado o en el arranque)
* Actualización interna: delegar el control a los propietarios de la aplicación
* Actualizaciones masivas a través de la automatización PowerCLI

En nuestro caso, utilizaremos la segunda opción para hacerlo de forma manual y controlada:

![Actualización manual][1]

# Referencias

* [VMware Tools 12.1.5 Release Notes](https://docs.vmware.com/en/VMware-Tools/12.1/rn/vmware-tools-1215-release-notes/index.html)

[1]: /assets/img/blog/2023-01-10_image_1.png "Actualización manual"
