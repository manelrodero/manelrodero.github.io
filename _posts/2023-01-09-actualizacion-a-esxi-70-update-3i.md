---
layout : post
blog-width: true
title: 'Actualización a ESXi 7.0 Update 3i'
date: '2023-01-09 14:31:27'
published: true
tags:
- Software
author:
  display_name: Manel Rodero
---

# Estado actual

El [VMSA-2022-0033](https://www.vmware.com/content/vmware/vmware-published-sites/us/security/advisories/VMSA-2022-0033.html) del pasado 13 de Diciembre de 2022 avisa de una vulnerabilidad **crítica** en **VMware ESXi 7.0** ([CVE-2022-31705](https://nvd.nist.gov/vuln/detail/CVE-2022-31705?idU=1)).

El servidor que tenemos que actualizar, un **Dell Inc. PowerEdge R340** con la _image profile_ `DEL-ESXi-702_17867351-A03 (Dell Inc.)`, indica que está ejecutando la versión **7.0 Update 2**.

Ahora bien, como en el _About_ del _VMware Host Client_ aparece la siguiente información:

```
Client version: 1.34.8
Client build number: 17417756
ESXi version: 7.0.2
ESXi build number: 17867351
```

realmente se trata de la _build_ **17867351** correspondiente a la versión [**7.0 Update 2a**](https://docs.vmware.com/en/VMware-vSphere/7.0/rn/vsphere-esxi-70u2a-release-notes.html) según las tablas del artículo [_Build numbers and versions of VMware ESXi/ESX_](https://kb.vmware.com/s/article/2143832).

# 7.0 Update 3i

En las _release notes_ de la versión [**7.0 Update 3i**](https://docs.vmware.com/en/VMware-vSphere/7.0/rn/vsphere-esxi-70u3i-release-notes.html) se indican las comprobaciones a realizar en aquellos servidores que:

* estén ejecutando una versión entre la 7.0 Update 2 y la 7.0 Update 3c
* y utilizen controladores Intel i40

Estas comprobaciones se pueden leer en la sección _What's New_ de las _release notes_ de [**VMware vCenter Server 7.0 Update 3c**](https://docs.vmware.com/en/VMware-vSphere/7.0/rn/vsphere-vcenter-server-70u3c-release-notes.html).

VMware proporciona un script `vSphere_upgrade_assessment.py` para comprobar si se requiere realizar alguna actuación previa a la actualización:

* [Using the vSphere_upgrade_assessment.py script](https://kb.vmware.com/s/article/87258)
* [vCenter 7.0 Update 3c Upgrade Precheck Demo](https://www.youtube.com/watch?v=xSA58Xzf8Wg)

> **Nota**: Estas actuaciones previas son importantes si el entorno está controlado desde un vCenter ya que será necesario actualizar los servidores ESXi antes del propio vCenter.

Aunque la manera típica de actualizar un ESXi sería utilizar [vSphere Lifecycle Manager](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere-lifecycle-manager.doc/GUID-74295A37-E8BB-4EB9-BFBA-47B78F0C570D.html), al ser éste un servidor _standalone_ se utilizarán comandos [ESXCLI](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.esxi.upgrade.doc/GUID-A4301ADA-8E02-459D-BF9D-0AD308DA5325.html).

Se puede descargar el fichero `VMware-ESXi-7.0U3i-20842708-depot.zip` con el _offline bundle_ de la _build_ **20842708** (8 de Diciembre de 2022) desde la sección de _patches_ de [VMware Customer Connect](https://customerconnect.vmware.com/patch/).

A continuación se subiría el fichero al servidor ESXi y después se ejecutaría el comando `esxcli software vib update -d /path/to/VMware-ESXi-7.0U3i-20842708-depot.zip`.

# DellEMC Custom Images

Otra opción sería descargar la versión personalizada por Dell desde la página de [drivers del PowerEdge R340](https://www.dell.com/support/home/en-us/product-support/product/poweredge-r340/drivers) filtrando por:

* Operating system: `VMware ESXi 7.0`
* Category: `Enterprise Solutions`

A fecha de hoy, únicamente se encuentra información sobre la versión [**7.0 U3, A09**](https://www.dell.com/support/home/en-us/drivers/driversdetails?driverid=gtfvg&oscode=xi70&productcode=poweredge-r340) (7 de Diciembre de 2022) basada en la _build_ **20328353**, es decir la **7.0 Update 3g** del 1 de Septiembre de 2022.

En esta misma página se explica que, desde hace tiempo, las imágenes _custom_ de Dell se descargan desde la propia web de VMware siguiendo estos pasos:

* Acceder al _dashboard_ de [VMware Customer Connect](https://customerconnect.vmware.com/dashboard)
* Iniciar sesión con nuestras credenciales
* Seleccionar `Products` > `All Products`
* Localizar el producto **VMware vSphere**
* Hacer click en `View Download Components`
* Seleccionar `7.0` en el desplegable de versiones
* Seleccionar la pestaña `Custom ISOs`
* Desplegar la sección `OEM Customized Installer CDs`
* Buscar la imagen requerida (en este caso la [**7.0 U3**](https://customerconnect.vmware.com/downloads/details?downloadGroup=OEM-ESXI70U3-DELLEMC&productId=974))
* Hacer click en `Go to downloads`

Desde aquí se pueden descargar dos versiones diferentes de la imagen _custom_:

* Dell Custom Image for ESXi 7.0 U3 Install CD (`*.iso`)
* Dell Custom Image for ESXi 7.0 U3 Offline Bundle (`*.zip`)

En este caso se descargaría el _offline bundle_ para realizar la actualización (`VMware-VMvisor-Installer-7.0.0.update03-20328353.x86_64-Dell_Customized-A09.zip`).

> **Nota**: Tal como se ha comentado anteriormente, se trata de la versión 7.0 Update 3g y no la 7.0 Update 3i que se requiere para solucionar la vulnerabilidad crítica del CVE-2022-31705.

# Procedimiento

El procedimiento para actualizar el servidor ESXi utilizando el [_offline bundle_ en formato `*.zip`](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.esxi.upgrade.doc/GUID-22A4B153-CB21-47B4-974E-2E5BB8AC6874.html) es el siguiente:

* Subir el fichero `*.zip` a un _datastore_ del servidor ESXi
* Parar las máquinas virtuales que se estén ejecutando en el servidor
* `Host` > `Actions` > `Enter maintenance mode`
* `Host` > `Actions` > `Services` > `Enable Secure Shell (SSH)`
* Conectarse al servidor mediante un cliente SSH
* Comprobar los VIBs instalados actualmente mediante el comando `esxcli software vib list`:

```
[root@esxi:~] esxcli software vib list
Name                           Version                              Vendor   Acceptance Level  Install Date
-----------------------------  -----------------------------------  -------  ----------------  ------------
bnxtnet                        218.0.38.0-1OEM.700.1.0.15843807     BCM      VMwareCertified   2021-06-15
bnxtroce                       218.0.16.0-1OEM.700.1.0.15843807     BCM      VMwareCertified   2021-06-15
dell-shared-perc8              06.806.92.00-1OEM.700.1.0.15843807   BCM      VMwareCertified   2021-06-15
dell-configuration-vib         7.0.0-A00                            DEL      PartnerSupported  2021-06-15
dellemc-osname-idrac           7.0.0-A00                            DEL      PartnerSupported  2021-06-15
brcmnvmefc                     12.8.329.0-1OEM.700.1.0.15843807     EMU      VMwareCertified   2021-06-15
lpfc                           12.8.340.12-1OEM.700.1.0.15843807    EMU      VMwareCertified   2021-06-15
i40en-ens                      1.2.7.0-1OEM.700.1.0.15843807        INT      VMwareCertified   2021-06-15
i40en                          1.10.9.0-1OEM.700.1.0.15525992       INT      VMwareCertified   2021-06-15
icen                           1.4.2.0-1OEM.700.1.0.15843807        INT      VMwareCertified   2021-06-15
igbn                           1.5.2.0-1OEM.700.1.0.15843807        INT      VMwareCertified   2021-06-15
ixgben-ens                     1.2.4.0-1OEM.700.1.0.15843807        INT      VMwareCertified   2021-06-15
ixgben                         1.8.9.0-1OEM.700.1.0.15525992        INT      VMwareCertified   2021-06-15
nmlx5-core                     4.19.70.1-1OEM.700.1.0.15525992      MEL      VMwareCertified   2021-06-15
nmlx5-rdma                     4.19.70.1-1OEM.700.1.0.15525992      MEL      VMwareCertified   2021-06-15
qlnativefc                     4.1.22.0-1OEM.700.1.0.15843807       Marvell  VMwareCertified   2021-06-15
qcnic                          2.0.58.0-1OEM.700.1.0.15843807       QLC      VMwareCertified   2021-06-15
qedentv                        3.40.30.1-1OEM.700.1.0.15843807      QLC      VMwareCertified   2021-06-15
qedf                           2.2.52.1-1OEM.700.1.0.15843807       QLC      VMwareCertified   2021-06-15
qedi                           2.19.51.0-1OEM.700.1.0.15843807      QLC      VMwareCertified   2021-06-15
qedrntv                        3.40.28.0-1OEM.700.1.0.15843807      QLC      VMwareCertified   2021-06-15
qfle3                          1.4.14.0-1OEM.700.1.0.15843807       QLC      VMwareCertified   2021-06-15
qfle3f                         2.1.19.0-1OEM.700.1.0.15843807       QLC      VMwareCertified   2021-06-15
qfle3i                         2.1.5.0-1OEM.700.1.0.15843807        QLC      VMwareCertified   2021-06-15
atlantic                       1.0.3.0-8vmw.702.0.0.17867351        VMW      VMwareCertified   2021-06-15
brcmfcoe                       12.0.1500.1-2vmw.702.0.0.17867351    VMW      VMwareCertified   2021-06-15
elxiscsi                       12.0.1200.0-8vmw.702.0.0.17867351    VMW      VMwareCertified   2021-06-15
elxnet                         12.0.1250.0-5vmw.702.0.0.17867351    VMW      VMwareCertified   2021-06-15
iavmd                          2.0.0.1152-1vmw.702.0.0.17867351     VMW      VMwareCertified   2021-06-15
irdman                         1.3.1.19-1vmw.702.0.0.17867351       VMW      VMwareCertified   2021-06-15
iser                           1.1.0.1-1vmw.702.0.0.17867351        VMW      VMwareCertified   2021-06-15
lpnic                          11.4.62.0-1vmw.702.0.0.17867351      VMW      VMwareCertified   2021-06-15
lsi-mr3                        7.716.03.00-1vmw.702.0.0.17867351    VMW      VMwareCertified   2021-06-15
lsi-msgpt2                     20.00.06.00-3vmw.702.0.0.17867351    VMW      VMwareCertified   2021-06-15
lsi-msgpt35                    17.00.02.00-1vmw.702.0.0.17867351    VMW      VMwareCertified   2021-06-15
lsi-msgpt3                     17.00.10.00-2vmw.702.0.0.17867351    VMW      VMwareCertified   2021-06-15
mtip32xx-native                3.9.8-1vmw.702.0.0.17867351          VMW      VMwareCertified   2021-06-15
ne1000                         0.8.4-11vmw.702.0.0.17867351         VMW      VMwareCertified   2021-06-15
nenic                          1.0.33.0-1vmw.702.0.0.17867351       VMW      VMwareCertified   2021-06-15
nfnic                          4.0.0.63-1vmw.702.0.0.17867351       VMW      VMwareCertified   2021-06-15
nhpsa                          70.0051.0.100-2vmw.702.0.0.17867351  VMW      VMwareCertified   2021-06-15
nmlx4-core                     3.19.16.8-2vmw.702.0.0.17867351      VMW      VMwareCertified   2021-06-15
nmlx4-en                       3.19.16.8-2vmw.702.0.0.17867351      VMW      VMwareCertified   2021-06-15
nmlx4-rdma                     3.19.16.8-2vmw.702.0.0.17867351      VMW      VMwareCertified   2021-06-15
ntg3                           4.1.5.0-0vmw.702.0.0.17867351        VMW      VMwareCertified   2021-06-15
nvme-pcie                      1.2.3.11-1vmw.702.0.0.17867351       VMW      VMwareCertified   2021-06-15
nvmerdma                       1.0.2.1-1vmw.702.0.0.17867351        VMW      VMwareCertified   2021-06-15
nvmxnet3-ens                   2.0.0.22-1vmw.702.0.0.17867351       VMW      VMwareCertified   2021-06-15
nvmxnet3                       2.0.0.30-1vmw.702.0.0.17867351       VMW      VMwareCertified   2021-06-15
pvscsi                         0.1-2vmw.702.0.0.17867351            VMW      VMwareCertified   2021-06-15
qflge                          1.1.0.11-1vmw.702.0.0.17867351       VMW      VMwareCertified   2021-06-15
rste                           2.0.2.0088-7vmw.702.0.0.17867351     VMW      VMwareCertified   2021-06-15
sfvmk                          2.4.0.2010-4vmw.702.0.0.17867351     VMW      VMwareCertified   2021-06-15
smartpqi                       70.4000.0.100-6vmw.702.0.0.17867351  VMW      VMwareCertified   2021-06-15
vmkata                         0.1-1vmw.702.0.0.17867351            VMW      VMwareCertified   2021-06-15
vmkfcoe                        1.0.0.2-1vmw.702.0.0.17867351        VMW      VMwareCertified   2021-06-15
vmkusb                         0.1-1vmw.702.0.0.17867351            VMW      VMwareCertified   2021-06-15
vmw-ahci                       2.0.9-1vmw.702.0.0.17867351          VMW      VMwareCertified   2021-06-15
clusterstore                   7.0.2-0.0.17867351                   VMware   VMwareCertified   2021-06-15
cpu-microcode                  7.0.2-0.0.17867351                   VMware   VMwareCertified   2021-06-15
crx                            7.0.2-0.0.17867351                   VMware   VMwareCertified   2021-06-15
elx-esx-libelxima.so           12.0.1200.0-4vmw.702.0.0.17867351    VMware   VMwareCertified   2021-06-15
esx-base                       7.0.2-0.0.17867351                   VMware   VMwareCertified   2021-06-15
esx-dvfilter-generic-fastpath  7.0.2-0.0.17867351                   VMware   VMwareCertified   2021-06-15
esx-ui                         1.34.8-17417756                      VMware   VMwareCertified   2021-06-15
esx-update                     7.0.2-0.0.17867351                   VMware   VMwareCertified   2021-06-15
esx-xserver                    7.0.2-0.0.17867351                   VMware   VMwareCertified   2021-06-15
gc                             7.0.2-0.0.17867351                   VMware   VMwareCertified   2021-06-15
loadesx                        7.0.2-0.0.17867351                   VMware   VMwareCertified   2021-06-15
lsuv2-hpv2-hpsa-plugin         1.0.0-3vmw.702.0.0.17867351          VMware   VMwareCertified   2021-06-15
lsuv2-intelv2-nvme-vmd-plugin  2.0.0-2vmw.702.0.0.17867351          VMware   VMwareCertified   2021-06-15
lsuv2-lsiv2-drivers-plugin     1.0.0-5vmw.702.0.0.17867351          VMware   VMwareCertified   2021-06-15
lsuv2-nvme-pcie-plugin         1.0.0-1vmw.702.0.0.17867351          VMware   VMwareCertified   2021-06-15
lsuv2-oem-dell-plugin          1.0.0-1vmw.702.0.0.17867351          VMware   VMwareCertified   2021-06-15
lsuv2-oem-hp-plugin            1.0.0-1vmw.702.0.0.17867351          VMware   VMwareCertified   2021-06-15
lsuv2-oem-lenovo-plugin        1.0.0-1vmw.702.0.0.17867351          VMware   VMwareCertified   2021-06-15
lsuv2-smartpqiv2-plugin        1.0.0-6vmw.702.0.0.17867351          VMware   VMwareCertified   2021-06-15
native-misc-drivers            7.0.2-0.0.17867351                   VMware   VMwareCertified   2021-06-15
vdfs                           7.0.2-0.0.17867351                   VMware   VMwareCertified   2021-06-15
vmware-esx-esxcli-nvme-plugin  1.2.0.42-1vmw.702.0.0.17867351       VMware   VMwareCertified   2021-06-15
vsan                           7.0.2-0.0.17867351                   VMware   VMwareCertified   2021-06-15
vsanhealth                     7.0.2-0.0.17867351                   VMware   VMwareCertified   2021-06-15
tools-light                    12.1.0.20219665-20295239             VMware   VMwareCertified   2022-09-02
```

* Determinar los perfiles existentes en el _depot_ mediante el comando `esxcli software sources profile list`:

```
[root@esxi:~] esxcli software sources profile list --depot=/vmfs/volumes/datastore1/Updates/VMware-VMvisor
-Installer-7.0.0.update03-20328353.x86_64-Dell_Customized-A09.zip
Name                       Vendor     Acceptance Level  Creation Time        Modification Time
-------------------------  ---------  ----------------  -------------------  -----------------
DEL-ESXi-703_20328353-A09  Dell Inc.  PartnerSupported  2022-10-13T16:10:10  2022-10-13T16:10:10
```

* Actualizar los VIBs existentes en el perfil mediante el comando `esxcli software profile update`:

```
[root@esxi:~] esxcli software profile update --depot=/vmfs/volumes/datastore1/Updates/VMware-VMvisor-Insta
ller-7.0.0.update03-20328353.x86_64-Dell_Customized-A09.zip --profile=DEL-ESXi-703_20328353-A09
Update Result
   Message: The update completed successfully, but the system needs to be rebooted for the changes to be effective.
   Reboot Required: true
   VIBs Installed: BCM_bootbank_bnxtnet_222.0.155.0-1OEM.700.1.0.15843807, BCM_bootbank_bnxtroce_222.0.155.0-1OEM.700.1.0.15843807,

   <...>

   VMware_bootbank_vsan_7.0.3-0.55.20328353, VMware_bootbank_vsanhealth_7.0.3-0.55.20328353
   VIBs Removed: BCM_bootbank_bnxtnet_218.0.38.0-1OEM.700.1.0.15843807, BCM_bootbank_bnxtroce_218.0.16.0-1OEM.700.1.0.15843807,

   <...>

   VMware_bootbank_vsan_7.0.2-0.0.17867351, VMware_bootbank_vsanhealth_7.0.2-0.0.17867351
   VIBs Skipped: BCM_bootbank_dell-shared-perc8_06.806.92.00-1OEM.700.1.0.15843807, VMware_locker_tools-light_12.0.0.19345655-20036586
```

> **Nota**: Al usar el comando `esxcli software profile update`, únicamente se actualizan los VIBs existentes en el perfil sin afectar a otros VIBs instalados en el servidor (ver _skipped_).

* **Reiniciar** el servidor ESXi para que se apliquen los cambios
* `Host` > `Actions` > `Services` > `Enable Secure Shell (SSH)`
* Conectarse al servidor mediante un cliente SSH
* Comprobar que los VIB se han actualizado correctamente:

```
[root@esxi:~] esxcli software vib list
Name                           Version                                Vendor  Acceptance Level  Install Date
-----------------------------  -------------------------------------  ------  ----------------  ------------
bnxtnet                        222.0.155.0-1OEM.700.1.0.15843807      BCM     VMwareCertified   2023-01-10
bnxtroce                       222.0.155.0-1OEM.700.1.0.15843807      BCM     VMwareCertified   2023-01-10
dell-shared-perc8              06.806.92.00-1OEM.700.1.0.15843807     BCM     VMwareCertified   2021-06-15
lsi-mr3                        7.722.02.00-1OEM.700.1.0.15843807      BCM     VMwareCertified   2023-01-10
lsi-msgpt35                    22.00.00.00-1OEM.700.1.0.15843807      BCM     VMwareCertified   2023-01-10
dell-configuration-vib         7.0.0-A02                              DEL     PartnerSupported  2023-01-10
dell-fac-dcui                  7.0.3-A02                              DEL     PartnerSupported  2023-01-10
dell-fist-util                 5.2.0.1-2022.10.07                     DEL     PartnerSupported  2023-01-10
dell-osname-idrac              7.0.0-A01                              DEL     PartnerSupported  2023-01-10
lpfc                           14.0.622.0-1OEM.700.1.0.15843807       EMU     VMwareCertified   2023-01-10
i40en                          2.3.4.0-1OEM.700.1.0.15843807          INT     VMwareCertified   2023-01-10
icen                           1.9.8.0-1OEM.702.0.0.17630552          INT     VMwareCertified   2023-01-10
igbn                           1.10.2.0-1OEM.700.1.0.15843807         INT     VMwareCertified   2023-01-10
irdman                         1.4.1.0-1OEM.700.1.0.15843807          INT     VMwareCertified   2023-01-10
ixgben-ens                     1.7.1.0-1OEM.700.1.0.15843807          INT     VMwareCertified   2023-01-10
ixgben                         1.13.1.0-1OEM.700.1.0.15843807         INT     VMwareCertified   2023-01-10
nmlx5-core                     4.21.71.101-1OEM.702.0.0.17630552      MEL     VMwareCertified   2023-01-10
nmlx5-rdma                     4.21.71.101-1OEM.702.0.0.17630552      MEL     VMwareCertified   2023-01-10
qlnativefc                     5.3.2.0-1OEM.703.0.0.18644231          MVL     VMwareCertified   2023-01-10
qcnic                          2.0.65.0-1OEM.700.1.0.15843807         QLC     VMwareCertified   2023-01-10
qedentv                        3.70.7.0-1OEM.700.1.0.15843807         QLC     VMwareCertified   2023-01-10
qedf                           2.30.8.3-1OEM.700.1.0.15843807         QLC     VMwareCertified   2023-01-10
qedi                           2.30.7.2-1OEM.700.1.0.15843807         QLC     VMwareCertified   2023-01-10
qedrntv                        3.70.7.0-1OEM.700.1.0.15843807         QLC     VMwareCertified   2023-01-10
qfle3                          1.4.35.0-1OEM.700.1.0.15843807         QLC     VMwareCertified   2023-01-10
qfle3f                         2.1.30.0-1OEM.700.1.0.15843807         QLC     VMwareCertified   2023-01-10
qfle3i                         2.1.12.0-1OEM.700.1.0.15843807         QLC     VMwareCertified   2023-01-10
atlantic                       1.0.3.0-8vmw.703.0.20.19193900         VMW     VMwareCertified   2023-01-10
brcmfcoe                       12.0.1500.2-3vmw.703.0.20.19193900     VMW     VMwareCertified   2023-01-10
elxiscsi                       12.0.1200.0-9vmw.703.0.20.19193900     VMW     VMwareCertified   2023-01-10
elxnet                         12.0.1250.0-5vmw.703.0.20.19193900     VMW     VMwareCertified   2023-01-10
iavmd                          2.7.0.1157-2vmw.703.0.20.19193900      VMW     VMwareCertified   2023-01-10
ionic-en                       16.0.0-16vmw.703.0.20.19193900         VMW     VMwareCertified   2023-01-10
iser                           1.1.0.1-1vmw.703.0.50.20036589         VMW     VMwareCertified   2023-01-10
lpnic                          11.4.62.0-1vmw.703.0.20.19193900       VMW     VMwareCertified   2023-01-10
lsi-msgpt2                     20.00.06.00-4vmw.703.0.20.19193900     VMW     VMwareCertified   2023-01-10
lsi-msgpt3                     17.00.12.00-1vmw.703.0.20.19193900     VMW     VMwareCertified   2023-01-10
mtip32xx-native                3.9.8-1vmw.703.0.20.19193900           VMW     VMwareCertified   2023-01-10
ne1000                         0.9.0-1vmw.703.0.50.20036589           VMW     VMwareCertified   2023-01-10
nenic                          1.0.33.0-1vmw.703.0.20.19193900        VMW     VMwareCertified   2023-01-10
nfnic                          4.0.0.70-1vmw.703.0.20.19193900        VMW     VMwareCertified   2023-01-10
nhpsa                          70.0051.0.100-4vmw.703.0.20.19193900   VMW     VMwareCertified   2023-01-10
nmlx4-core                     3.19.16.8-2vmw.703.0.20.19193900       VMW     VMwareCertified   2023-01-10
nmlx4-en                       3.19.16.8-2vmw.703.0.20.19193900       VMW     VMwareCertified   2023-01-10
nmlx4-rdma                     3.19.16.8-2vmw.703.0.20.19193900       VMW     VMwareCertified   2023-01-10
ntg3                           4.1.7.0-0vmw.703.0.20.19193900         VMW     VMwareCertified   2023-01-10
nvme-pcie                      1.2.3.16-1vmw.703.0.20.19193900        VMW     VMwareCertified   2023-01-10
nvmerdma                       1.0.3.5-1vmw.703.0.20.19193900         VMW     VMwareCertified   2023-01-10
nvmetcp                        1.0.0.1-1vmw.703.0.35.19482537         VMW     VMwareCertified   2023-01-10
nvmxnet3-ens                   2.0.0.22-1vmw.703.0.20.19193900        VMW     VMwareCertified   2023-01-10
nvmxnet3                       2.0.0.30-1vmw.703.0.20.19193900        VMW     VMwareCertified   2023-01-10
pvscsi                         0.1-4vmw.703.0.20.19193900             VMW     VMwareCertified   2023-01-10
qflge                          1.1.0.11-1vmw.703.0.20.19193900        VMW     VMwareCertified   2023-01-10
rste                           2.0.2.0088-7vmw.703.0.20.19193900      VMW     VMwareCertified   2023-01-10
sfvmk                          2.4.0.2010-6vmw.703.0.20.19193900      VMW     VMwareCertified   2023-01-10
smartpqi                       70.4149.0.5000-1vmw.703.0.20.19193900  VMW     VMwareCertified   2023-01-10
vmkata                         0.1-1vmw.703.0.20.19193900             VMW     VMwareCertified   2023-01-10
vmkfcoe                        1.0.0.2-1vmw.703.0.20.19193900         VMW     VMwareCertified   2023-01-10
vmkusb                         0.1-7vmw.703.0.50.20036589             VMW     VMwareCertified   2023-01-10
vmw-ahci                       2.0.11-1vmw.703.0.20.19193900          VMW     VMwareCertified   2023-01-10
bmcal                          7.0.3-0.55.20328353                    VMware  VMwareCertified   2023-01-10
cpu-microcode                  7.0.3-0.55.20328353                    VMware  VMwareCertified   2023-01-10
crx                            7.0.3-0.55.20328353                    VMware  VMwareCertified   2023-01-10
elx-esx-libelxima.so           12.0.1200.0-4vmw.703.0.20.19193900     VMware  VMwareCertified   2023-01-10
esx-base                       7.0.3-0.55.20328353                    VMware  VMwareCertified   2023-01-10
esx-dvfilter-generic-fastpath  7.0.3-0.55.20328353                    VMware  VMwareCertified   2023-01-10
esx-ui                         1.43.8-19798623                        VMware  VMwareCertified   2023-01-10
esx-update                     7.0.3-0.55.20328353                    VMware  VMwareCertified   2023-01-10
esx-xserver                    7.0.3-0.55.20328353                    VMware  VMwareCertified   2023-01-10
esxio-combiner                 7.0.3-0.55.20328353                    VMware  VMwareCertified   2023-01-10
gc                             7.0.3-0.55.20328353                    VMware  VMwareCertified   2023-01-10
loadesx                        7.0.3-0.55.20328353                    VMware  VMwareCertified   2023-01-10
lsuv2-hpv2-hpsa-plugin         1.0.0-3vmw.703.0.20.19193900           VMware  VMwareCertified   2023-01-10
lsuv2-intelv2-nvme-vmd-plugin  2.7.2173-1vmw.703.0.20.19193900        VMware  VMwareCertified   2023-01-10
lsuv2-lsiv2-drivers-plugin     1.0.0-12vmw.703.0.50.20036589          VMware  VMwareCertified   2023-01-10
lsuv2-nvme-pcie-plugin         1.0.0-1vmw.703.0.20.19193900           VMware  VMwareCertified   2023-01-10
lsuv2-oem-dell-plugin          1.0.0-1vmw.703.0.20.19193900           VMware  VMwareCertified   2023-01-10
lsuv2-oem-hp-plugin            1.0.0-1vmw.703.0.20.19193900           VMware  VMwareCertified   2023-01-10
lsuv2-oem-lenovo-plugin        1.0.0-1vmw.703.0.20.19193900           VMware  VMwareCertified   2023-01-10
lsuv2-smartpqiv2-plugin        1.0.0-8vmw.703.0.20.19193900           VMware  VMwareCertified   2023-01-10
native-misc-drivers            7.0.3-0.55.20328353                    VMware  VMwareCertified   2023-01-10
trx                            7.0.3-0.55.20328353                    VMware  VMwareCertified   2023-01-10
vdfs                           7.0.3-0.55.20328353                    VMware  VMwareCertified   2023-01-10
vmware-esx-esxcli-nvme-plugin  1.2.0.44-1vmw.703.0.20.19193900        VMware  VMwareCertified   2023-01-10
vsan                           7.0.3-0.55.20328353                    VMware  VMwareCertified   2023-01-10
vsanhealth                     7.0.3-0.55.20328353                    VMware  VMwareCertified   2023-01-10
tools-light                    12.1.0.20219665-20295239               VMware  VMwareCertified   2022-09-02
```

> **Nota**: No se han actualizado los VIBs `dell-shared-perc8` (un dispositivo [_deprecated_ en ESXi 8.0](https://kb.vmware.com/s/article/88172)) y `tools-light` (correspondiente a las VMware Tools que se actualizan de forma independiente).

* Comprobar las firmas de los VIBs instalados, necesarias para activar el [_Secure Boot_](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.esxi.upgrade.doc/GUID-DD075A38-8900-459F-BD9D-69DC87CCE11B.html), mediante el comando `/usr/lib/vmware/secureboot/bin/secureBoot.py -c`:

```
[root@esxi:~] /usr/lib/vmware/secureboot/bin/secureBoot.py -c
Secure boot can be enabled: All vib signatures verified. All tardisks validated. All acceptance levels validated
```

* `Host` > `Actions` > `Services` > `Disable Secure Shell (SSH)`
* `Host` > `Actions` > `Exit maintenance mode`
* Poner en marcha las VMs del servidor

El servidor ESXi debería indicar que se está ejecutando la versión **7.0 Update 3** y en el _About_ del _VMware Host Client_ aparecer la siguiente información:

```
Client version: 1.43.8
Client build number: 19798623
ESXi version: 7.0.3
ESXi build number: 20328353
```

# Referencias

* [VMware vSphere ESXi 7.x on Dell EMC PowerEdge Systems](https://dl.dell.com/topicspdf/vmware-esxi-7x_reference-guide_en-us.pdf)
* [Dell VMware ESXi 7.0.x Manuals](https://www.dell.com/support/home/product-support/product/vmware-esxi-7.x/docs)
