---
layout : post
blog-width: true
css: /assets/css/youtube.css
title: 'Notas del Cloud Imaging Day (FWSMUG)'
date: '2020-11-16 08:42:48'
published: true
tags:
- OSD
- PowerShell
- MDT
author:
  display_name: Manel Rodero
share-img: "/assets/img/blog/2020-11-16_image_2.png"  
thumbnail-img: ""
---

El pasado 12 de noviembre de 2020 se celebró el evento **Cloud Imaging Day** del [FWSMUG](https://twitter.com/fwsmug) en el que [Johan Arwidmark](https://twitter.com/jarwidmark) y [Mikael Nystrom](https://twitter.com/mikael_nystrom), moderados por [Ami Arwidmark](https://twitter.com/AArwidmark), presentaron su _framework_ **[PowerShell Deployment Extension Kit](https://github.com/FriendsOfMDT/PSD)** que permite desplegar un equipo a través de Internet mediante HTTP/HTTPS.

El evento se realizó a través de la plataforma GoToWebinar y tuvo una duración aproximada de 3h. Unos días más tarde se publicaron las grabaciones del mismo en el [canal de YouTube de Ami Arwidmark](https://www.youtube.com/channel/UCHagX0LRoqzhI8AfD37SD3g):

* [FWSMUG Nov 2020 Webinar - Cloud Imaging Day #1](https://www.youtube.com/watch?v=R6-2ybv2hcs)
* [FWSMUG Nov 2020 Webinar - Cloud Imaging Day #2](https://www.youtube.com/watch?v=nv9z4vhCAaE)
* [FWSMUG Nov 2020 Webinar - Cloud Imaging Day #3](https://www.youtube.com/watch?v=8wmdNKPkr_I)

Este artículo es un resumen de las ideas y conceptos explicados durante el evento en el orden en el que han ido apareciendo. No pretende ser algo estructurado y con una buena redacción, sino unas notas rápidas con todo aquello que será interesante investigar.

## Gestión de los equipos

El evento comenzó hablando de las tres posibilidades que hay actualmente para que un equipo pertenezca a un entorno gestionado:

* [Active Directory Domain Services](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview) (ADDS)
* [Azure Active Directory](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-whatis) (AAD o Azure AD)
* [Microsoft Intune](https://docs.microsoft.com/en-us/mem/intune/fundamentals/what-is-intune)

Actualmente muchas empresas comienzan a desplegar sus equipos en [Azure](https://docs.microsoft.com/en-us/azure/) o [AWS](https://docs.aws.amazon.com/) (es decir, en la nube) y aquí cobran importancia los siguientes conceptos:

* Microsoft Intune
* [Windows Autopilot](https://docs.microsoft.com/en-us/mem/autopilot/windows-autopilot)
* [Microsoft Graph](https://docs.microsoft.com/en-us/graph/overview)
* [Cloud Management Gateway](https://docs.microsoft.com/en-us/mem/configmgr/core/clients/manage/cmg/overview) (CMG)

**Windows Autopilot** es una tecnología que permite automatizar la colocación de un equipo bajo la gestión de Microsoft Intune. Los equipos salen de fábrica con un Windows 10 preparado para este proceso y pueden ser enviados directamente a los usuarios finales. Cuando éstos inician sesión con su cuenta de la organización, el equipo se configura y se instalan las aplicaciones que se hayan indicado.

También existe la posibilidad de realizar una pre-instalación de las mismas por el fabricante o por el equipo de IT de la empresa. Esta opción se llama [**pre-provisoning**](https://docs.microsoft.com/en-us/mem/autopilot/pre-provision) aunque antes se llamaba _white glove_.

El siguiente vídeo muestra los diferentes escenarios que soporta Windows Autopilot:

* [Microsoft Windows Autopilot deployment scenarios](https://youtu.be/pmRLEBK55PU)

**Microsoft Graph** permite automatizar ciertas tareas desde el lado servidor. Se crea una aplicación en Microsoft Intune que tiene permisos para utilizar Microsoft Graph y realizar ciertas operaciones en Azure AD (por ejemplo añadir usuarios y/o máquinas).

La asignación de **aplicaciones** y **perfiles** en Azure AD (o Microsoft Intune) se realiza a través de los **grupos**. En un Active Directory tradicional se realizaría mediante _Group Policies_ y en Config Manager mediante colecciones.

{: .box-warning}
**Atención:** La persona que gestione Microsoft Intune en la organización **no es necesario que sea _Global Admin_** aunque tiene que poder [**crear grupos**](https://docs.microsoft.com/en-us/microsoft-365/admin/add-users/about-admin-roles) para que todo funcione correctamente.

La administración de grupos y máquinas se realiza desde el **Microsoft Endpoint Manager Admin Center** ([https://endpoint.microsoft.com/](https://endpoint.microsoft.com/)) accediendo a las siguientes opciones:

* Groups
* Devices > Windows > Windows devices

La [**Enrollment Status Page**](https://docs.microsoft.com/en-us/mem/intune/enrollment/windows-enrollment-status) es una página que se muestra a los usuarios durante la configuración inicial del dispositivo y durante el primer inicio de sesión. Si está asignada, los usuarios pueden ver el progreso de la configuración de las aplicaciones y perfiles asignados a su dispositivo. La página asignada por defecto '_All users and all devices_' no muestra ese progreso.

Los [**Deployment Profiles**](https://docs.microsoft.com/en-us/mem/autopilot/enrollment-autopilot) permiten personalizar la experiencia _out-of-box_ de los dispositivos. Por defecto no hay creado ninguno y habrá que hacerlo cuando se comience a trabajar con Windows Autopilot.

[![Windows Enrollment][1]](https://endpoint.microsoft.com/#blade/Microsoft_Intune_DeviceSettings/DevicesWindowsMenu/windowsEnrollment)

{: .box-note}
**Nota**: Johan utiliza [Remote Desktop Manager Free](https://remotedesktopmanager.com/home/downloadfree) mientras que Mikael continúa usando [Remote Desktop Connection Manager (RDCMan)](https://techcommunity.microsoft.com/t5/exchange-team-blog/introducing-remote-desktop-connection-manager-rdcman-2-2/ba-p/592989) que ha sido [descontinuado por Microsoft](https://blog.devolutions.net/2020/04/microsoft-discontinues-remote-desktop-connection-manager-rdcman-invitation-to-try-remote-desktop-manager-rdm) debido al [CVE-2020-0765](https://nvd.nist.gov/vuln/detail/CVE-2020-0765).

## PowerShell Deployment Extension Toolkit

PSD es una extensión realizada en PowerShell que está pensada como sustitución del código `*.vbs` que se utiliza en [**Microsoft Deployment Toolkit (MDT)**](https://docs.microsoft.com/en-us/windows/deployment/deploy-windows-mdt/get-started-with-the-microsoft-deployment-toolkit). Actualmente permite desplegar un equipo por HTTP/HTTPS además del tradicional uso de UNC. Al ser extensible, podrían programarse diferentes proveedores para descargar mediante FTP, P2P, etc.

Johan muestra un ejemplo de uso en el que una VM inicia mediante PXE y descarga el NBP `boot/x64/snponly_x66.efi` (basado en el software [**iPXE Anywhere**](https://2pintsoftware.com/products/ipxeanywhere/) de [2Pint Software](https://2pintsoftware.com/)) y, una vez autenticado mediante usuario/contraseña, procede a descargar la _boot image_ `PSD_LiteTouch.wim` desde Azure.

La forma de iniciar esta imagen puede ser muy variada:

* Desde un fichero ISO en máquinas virtuales
* Mediante un USB (aunque no se recomienda porque se quedan obsoletos rápidamente)
* Desde la BIOS/UEFI
* Con un teléfono Android/iOS

Las BIOS más modernas incorporan algún modo para descargar la imagen desde Internet. Por ejemplo, HP incluye [HP Sure Recover](http://h10032.www1.hp.com/ctg/Manual/c06579216) en sus BIOS y mediante `F11` puede descargar una imagen desde la URL que se haya configurado previamente mediante PowerShell. Ésto se puede realizar mediante un script similar al que se utiliza para aprovisionar el chip TPM.

Cuando alguien pregunta por la seguridad (autenticación, contraseñas, etc.) en la _boot image_, Mikael explica que usan [**RestPS**](https://deploymentbunny.com/2020/11/15/nice-to-know-running-restps-as-a-service/), un _web service_ realizado en PowerShell que será quien, mediante Microsoft Graph, haga las operacions críticas en Azure AD. De esta manera se evitan las credenciales en los _scripts_.

Si se quiere proteger aún más la comunicación, el uso de **certificados de cliente** sería una buena solución.

El uso de HTTP/HTTPS para descargar es igual o más rápido que la descarga mediante SMB desde una ruta UNC. Además hay otras ventajas:

* Los paquetes de _drivers_ y aplicaciones se pueden comprimir mediante ZIP (habrá que extraerlos) o WIM (habrá que montar y desmontar)
* Se puede utilizar P2P para descargar. Se aprovecha la [**Branch Cache**](https://docs.microsoft.com/en-us/windows-server/networking/branchcache/branchcache) de los equipos vecinos usando la tecnología de 2Pint Software

{: .box-note}
**Nota**: 2Pint Software tiene un producto gratuito llamado [OSD Toolkit](
https://2pintsoftware.com/products/osd-toolkit/) que permite usar Microsoft BITS y Branch Cache en el entorno Windows PE para acelerar los despliegues.

Otra ventaja de los formatos anteriores es que aprovechan muchísimo la **deduplicación** (y, por tanto, reducen el espacio en disco ocupado por las aplicaciones y los controladores).

PSD puede ejecutar acciones previas mediante un menú simple definido en un fichero `*.xml`. El script encargado de hacerlo es [`PSDPrestart.ps1`](https://github.com/FriendsOfMDT/PSD/blob/master/Scripts/PSDPrestart.ps1).

También es posible utilizar credenciales para proteger la Task Sequence. Antes de comenzarla se presenta un _wizard_ al usuario para seleccionar diferentes parámetros (nombre del equipo, contraseña del administrador, grupo al que se añadirá, etc.).

Todo ésto está basado en los ficheros tradicionales `customsettings.ini` (está en el _share_) y `bootstrap.ini` (está en la _boot image_). Se puede encontrar más información sobre estos ficheros en [Toolkit Reference for MDT](https://docs.microsoft.com/en-us/mem/configmgr/mdt/toolkit-reference).

La gracia de usar PSD Extension Kit es que está basado completamente en una instalación normal de MDT:

* Se usa Deployment Workbench en el servidor
* Se usan algunas partes de MDT en el cliente para acceder a los servicios de MDT desde PowerShell (módulos `*.psd1` y `*.dll` en el directorio `Control`)
* Descarga los ficheros a la caché usando HTTP/HTTPS (o UNC si así se quiere)

Otra ventaja de usar HTTP/HTTPS además de la rapidez explicada anteriormente es que es un protocolo sin estado y además fácil de hacer pasar por el _firewall_.

La instalación de un servidor para utilizar PSD es la misma que para usar MDT. Se tiene que instalar MDT y despues el ADK (atención al parche para las últimas versiones de Windows 10). Además se instalará IIS ya que se utiliza WebDav para acceder al _deployment share_.

Una vez instalado, se crea un _deployment share_ en el que habrá una nueva plantilla para desplegar usando PSD. Después habrá que modificar el `bootstrap.ini`, crear las reglas y añadir el contenido como se hace habitualmente en MDT.

Mikael explica que lo están usando en producción para desplegar en las siguientes situaciones:

* Despliegue de [**Privileged Access Workstations** (PAW)](https://docs.microsoft.com/en-us/windows-server/identity/securing-privileged-access/privileged-access-workstations) donde no puede haber agentes externos para gestionarla
* Resolución de incidentes de seguridad (p.ej. ransomware)
* Despliegue a ubicaciones problemáticas
* Despliegue a través de la WAN
* Despliegue a través de la LAN
* Integrado con:
  * Azure AD Join
  * AD Join
  * Intune/Autopilot (recuperación de equipos)
  * ConfigMgr

{: .box-warning}
**Atención**: Se recomienda no tocar mucho el _script_ [`PSDStart.ps1`](https://github.com/FriendsOfMDT/PSD/blob/master/Scripts/PSDStart.ps1) ya que es el corazón de todo el módulo. Tampoco es recomendable tocar [`PSDGather.ps1`](https://github.com/FriendsOfMDT/PSD/blob/master/Scripts/PSDGather.ps1) ya que es el encargado de obtener información sobre el equipo y colocarla en variables que puedan ser consumidas desde el resto de scripts. Los demás no hay problemas por modificarlos.

Johan y Mikael piden voluntarios para probar el módulo PSD. [Damien Van Robaeys](https://twitter.com/syst_and_deploy) y [Jérôme Bezet-Torres](https://twitter.com/JM2K69) están colaborando para mejorar la interfaz gráfica del _wizard_ que se ejecuta en la _boot image_. 

[![PSD Wizard (PSD Extension for MDT)][2]](https://deploymentresearch.com/cloud-os-deployment-part-2-bare-metal-deployment-via-mdt-from-the-cloud/)

Una limitación actual del módulo PSD es que únicamente se puede utilizar para despliegues _bare metal_ y no se puede usar para _in place upgrades_ aunque está previsto para versiones futuras en cuanto tengan algo de tiempo.

Si tenemos un _deployment share_ tradicional, se pueden copiar las aplicaciones y los controladores para poder usarlos en un despliegue PSD.

Han surgido preguntas acerca de la instalación de controladores desde la línea de comandos usando las herramientas del fabricante (Dell Command Update, [HP Image Assistant y HP CMSL](https://developers.hp.com/pt/node/12584), etc.). Tanto Mikael como Johan son partidarios de tener un control más exhaustivo sobre los controladores y no obtenerlos de esa manera. Tampoco aconsejan hacerlo desde Windows Update for Business.

Claro que, si no se van a probar antes, lo mejor es no deshabilitar la _Business Store_ y que se descarguen desde allí. Algunas aplicaciones necesarias para que un controlador funcione se están comenzando a distribuir mediante _packages_ `*.appx`. Si se hace de esta manera, se actualizarán automáticamente siempre y cuando no se haya bloqueado.

Tanto en este entorno como en Config Manager, se recomienda realizar una **gestión moderna** de [los drivers](https://msendpointmgr.com/modern-driver-management/) y de [las BIOS](https://msendpointmgr.com/modern-bios-management/) usando **Driver Automation Tool** de [Nickolaj Andersen](https://twitter.com/NickolajA) y [Maurice Daly](https://twitter.com/Modaly_IT).

## Cloud Management Gateway

En la parte final de la sesión, Johan explica cómo usar **Cloud Management Gateway** para gestionar equipos que están fuera de la red local. Hizo un estudio que publicó en su blog con los costes que tenía este producto y solía ser muy pequeño, sobretodo si se utilizan tecnologías P2P para mejorarlo.

Se utiliza CMG para **gestionar** y para **desplegar** los equipos. Ahora bien, si el equipo está unido a un AD, tiene que poder comunicarse con él. Se pierde el sentido del uso de CMG y lo más aconsejable es sacarlo del AD y unirlo a un Azure AD.

Config Manager ha ido evolucionando en cuanto a lo que podía hacer a través de un CMG:

* Antiguamente: únicamente TS IPU
* Versión 2006: se incluyeron despliegues _bare metal_
* Versión 2009 TP: ya no hace falta contactar con un MP y se descarga directamente desde el CMG

Aunque configurar CMG se hace en pocas horas, la obtención de todo lo necesario (certificados, contactar con el administrador global, etc.) puede hacer que sean días. O incluso semanas si se quieren validar todos los escenarios.

Es mejor utilizar **Enhanced HTTP** para la comunicación entre clientes y servidor y que todos estén en la versión 2006 o superior. No se aconseja el uso de PKI ya que complica un poco las cosas.

Es necesario comprender conceptos como **VPN Split Tunnel** para no tener costes con las descargas. De todos modos el uso de VPN está disminuyendo a medida que las organizaciones mueven los equipos a Azure AD o a equipos _standalone_.

Johan tiene 3 posts relaciones con el tema que recomienda leer para profundizar y entender los conceptos:

* [Cloud OS Deployment, Part 1 – Running MDT Task Sequences from Microsoft Intune](https://deploymentresearch.com/cloud-os-deployment-part-1-running-mdt-task-sequences-from-microsoft-intune/)
* [Cloud OS Deployment, Part 2 – Bare Metal Deployment via MDT from the Cloud](https://deploymentresearch.com/cloud-os-deployment-part-2-bare-metal-deployment-via-mdt-from-the-cloud/)
* [Cloud OS Deployment, Part 3 – Bare Metal Deployment via ConfigMgr with Content from the Cloud](https://deploymentresearch.com/cloud-os-deployment-part-3-bare-metal-deployment-via-configmgr-with-content-from-the-cloud/)

{: .box-note}
**Nota**: Es recomendable usar `cmtrace.exe` para leer `x:\windows\temp\smstslog\smsts.log` mientras se depura una _task sequence_.

Respecto a acelerar el despliegue comprimiendo los _drivers_ y las aplicaciones también tiene un post interesante:

* [Speed Up Driver Package Downloads for ConfigMgr OSD](https://deploymentresearch.com/speed-up-driver-package-downloads-for-configmgr-osd/)

{: .box-warning}
**Atención**: Cuando se utilice Driver Automation Tool es muy importante utilizar `ConfigMgr - Standard Pkg` en lugar de la opción "antigua" `ConfigMgr - Driver Pkg`.

Si se quiere mejorar las tareas de actualización IPU (_in place upgrade_) es necesario precargar los controladores en el equipo antes de comenzarla y evitar así esperas innecesarias a los usuarios:

* [Improving the ConfigMgr Inplace-Upgrade Task Sequence](https://deploymentresearch.com/improving-the-configmgr-inplace-upgrade-task-sequence/)

Al final de la sesión se comenta que MDT es una solución que lleva mucho tiempo en el mercado y se cree que aún seguirá muchos años más porque todo el mundo necesita desplegar SO. Se usará MDT para desplegar el SO, instalar los agentes de ConfigMgr o de Intune y comenzar a gestionarlo.

## Extras

Otros artículos relacionados con PSD/MDT:

* [OS Deployment – PowerShell Deployment Extension for MDT](https://deploymentbunny.com/2019/05/10/os-deployment-powershell-deployment-extension-for-mdt/) by [Mikael Nystrom](https://twitter.com/mikael_nystrom)
* [PowerShell Deployment – Getting Started with the PSD Hydration Kit](https://msendpointmgr.com/2019/05/21/powershell-deployment-getting-started-with-the-psd-hydration-kit/) by [Donna Ryan](https://twitter.com/TheNotoriousDRR)
* [Imaging from the Cloud – How to setup Powershell Deployment Extension (PSD) for MDT with HTTPS](https://brookspeppin.com/2020/06/26/how-to-setup-powershell-deployment-extension-for-mdt-with-https/) by [Brooks Peppin](https://twitter.com/brookspeppin)

[1]: /assets/img/blog/2020-11-16_image_1.png "Windows Enrollment"
[2]: /assets/img/blog/2020-11-16_image_2.png "PSD Wizard (PSD Extension for MDT)"