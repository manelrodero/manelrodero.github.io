---
layout: page
title: Windows 10
blog-width: true
---

Esta página contiene tablas y otros recursos de utilidad para los administradores del sistema operativo [Windows 10](https://docs.microsoft.com/en-us/windows/windows-10/){:target="_blank"}.

| [Actualizaciones](#actualizaciones) | [Ciclo de vida](#lifecycle) | [ADMX](#admx) | [RSAT](#rsat) | [Security Baselines](#baselines) | [Windows ADK](#adk) | [Descargas](#downloads) | [Artículos](#posts) |

# <a name="actualizaciones"></a>[Actualizaciones](https://aka.ms/WindowsUpdateHistory){:target="_blank"}

{: .box-note}
**Nota**: El LCU indicado es el correspondiente al _Patch Tuesday_. Desde la versión 2004 también incluye el [SSU](https://docs.microsoft.com/en-us/windows/deployment/update/servicing-stack-updates).

| Versión | OS Build | Fecha actualización | Cumulative Update (CU) | Servicing Stack Update (SSU) |
| --- | --- | --- | --- |
| 21H1 | 19043.985 | 11 Mayo 2021 | [KB5003173](https://support.microsoft.com/en-us/help/5003173){:target="_blank"} | - |
| 20H2 | 19042.985 | 11 Mayo 2021 | [KB5003173](https://support.microsoft.com/en-us/help/5003173){:target="_blank"} | - |
| 2004 | 19041.985 | 11 Mayo 2021 | [KB5003173](https://support.microsoft.com/en-us/help/5003173){:target="_blank"} | - |
| 1909 | 18363.1556 | 11 Mayo 2021 | [KB5003169](https://support.microsoft.com/en-us/help/5003169){:target="_blank"} | [KB5003244](https://support.microsoft.com/en-us/help/5003244){:target="_blank"} |

<!-- https://en.wikipedia.org/wiki/Windows_10_version_history -->

# <a name="lifecycle"></a>[Ciclo de vida](https://support.microsoft.com/en-us/help/13853/windows-lifecycle-fact-sheet){:target="_blank"} (_lifecycle_)

{: .box-note}
**Nota**: El [final de soporte](https://docs.microsoft.com/en-us/windows/release-health/release-information) indicado es para las versiones **Enterprise** y **Education**.

| Versión | Compilación | Codename | Marketing | Fecha de liberación | Final de soporte |
| --- | --- | --- | --- | --- | --- |
| 21H1 | 19043 | 21H1 | May 2021 Update | 18 Mayo 2021 | 13 Diciembre 2022 |
| 20H2 | 19042 | 20H2 | October 2020 Update | 20 Octubre 2020 | 9 Mayo 2023 |
| 2004 | 19041 | 20H1 | May 2020 Update | 27 Mayo 2020 | 14 Diciembre 2021 |
| 1909 | 18363 | 19H2 | November 2019 Update | 12 Noviembre 2019 | 10 Mayo 2022 |
| ~~1903~~ | 18362 | 19H1 | May 2019 Update | 21 Mayo 2019 | 8 Diciembre 2020 |
| ~~1809~~ | 17763 | Redstone 5 (RS5) | October 2018 Update| 13 Noviembre 2018 | 11 Mayo 2021 |
| ~~1803~~ | 17134 | Redstone 4 (RS4) | April 2018 Update | 30 Abril 2018 | 11 Mayo 2021 |
| ~~1709~~ | 16299 | Redstone 3 (RS3) | Fall Creators Update | 17 Octubre 2017 | 14 Abril 2020 |
| ~~1703~~ | 15063 | Redstone 2 (RS2) | Creators Update | 5 Abril 2017 | 8 Octubre 2019 |
| ~~1607~~ | 14393 | Redstone 1 (RS1) | Anniversary Update | 2 Agosto 2016 | 9 Abril 2019 |
| ~~1511~~ | 10586 | Threshold 2 (TH2) | November Update | 10 Noviembre 2015 | 10 Octubre 2017 |
| ~~1507~~ | 10240 | Threshold 1 (TH1) | | 29 Julio 2015 | 9 Mayo 2017 |

# <a name="admx"></a>Administrative Templates (*.ADMX)

Las políticas de grupo (GPO) permiten configurar muchísimos aspectos del sistema operativo. Estas políticas de grupo se pueden crear a partir de las plantillas administrativas almacenadas localmente o en un [**almacén central**](https://docs.microsoft.com/en-us/troubleshoot/windows-client/group-policy/create-and-manage-central-store) en el **controlador de dominio**.

{: .box-note}
**Nota**: Las plantillas administrativas son acumulativas y los ficheros `*.admx` y `*.adml` (lenguajes) se pueden sobreescribir con las versiones más recientes.

{: .box-note}
**Nota**: Para cada plantilla administrativa suele existir un fichero **Excel** con la referencia de todas las configuraciones incluidas en la misma. En algunos casos es una descarga independiente y en otros está incluida en las _security baselines_.

| Build | ADMX | Fecha de publicación | XLSX | Fecha de publicación |
| --- | --- | --- | --- | --- |
| 21H1 | [1.0](https://www.microsoft.com/en-us/download/details.aspx?id=103124) | 18 Mayo 2021 | [1.0](https://www.microsoft.com/en-us/download/details.aspx?id=103125) | 18 Mayo 2021 |
| 20H2 | [1.0](https://www.microsoft.com/en-us/download/details.aspx?id=102157) | 8 Octubre 2020 | [1.0](https://www.microsoft.com/en-us/download/details.aspx?id=102158) | 8 Octubre 2020 |
| 2004 | [1.0](https://www.microsoft.com/en-us/download/details.aspx?id=101445) | 10 Junio 2020 | [1.0](https://www.microsoft.com/en-us/download/details.aspx?id=101451) | 20 Junio 2020 |
| 1909 | [1.0](https://www.microsoft.com/en-us/download/details.aspx?id=100591) | 18 Diciembre 2019 | | |
| 1903 | [3.0](https://www.microsoft.com/en-us/download/details.aspx?id=58495) | 18 Diciembre 2019 | | |
| 1809 | [2.0](https://www.microsoft.com/en-us/download/details.aspx?id=57576) | 11 Diciembre 2019 | [1809](https://www.microsoft.com/en-us/download/details.aspx?id=57464) | 23 Octubre 2018 |
| 1803 | [2.0](https://www.microsoft.com/en-us/download/details.aspx?id=56880) | 13 Julio 2018 | | |
| 1709 | [1.0](https://www.microsoft.com/en-us/download/details.aspx?id=56121) | 18 Octubre 2017 | | |
| 1703 | [2.0](https://www.microsoft.com/en-us/download/details.aspx?id=55080) | 29 Junio 2017 | | |
| 1607 | [2.0](https://www.microsoft.com/en-us/download/details.aspx?id=53430) | 8 Enero 2017 | | |
| 1511 | [1.0](https://www.microsoft.com/en-us/download/details.aspx?id=48257) | 17 Noviembre 2015 | | |
| 1507 | [1.0](https://www.microsoft.com/en-us/download/details.aspx?id=48257) | 17 Noviembre 2015 | | |

# <a name="rsat"></a>Remote Server Administration Tool (RSAT)

Las herramientas de administración remota del servidor para Windows 10 1803 o anteriores se tienen que descargar desde Microsoft. A partir de Windows 10 1809 se instalan como una Feature on Demand.

{: .box-note}
**Nota**: Se ha documentado el proceso en el post "[Añadir RSAT en Windows 10 1809](/blog/anadir-rsat-en-windows-10-1809)" publicado en el blog. Existe un [script en PowerShell](https://github.com/imabdk/Powershell/blob/master/Install-RSATv1809v1903v1909v2004v20H2.ps1) creado por [Martin Bengtsson](https://www.imab.dk/) que se puede usar para instalar más fácilmente, incluso desde [Software Center](https://www.imab.dk/deploy-rsat-remote-server-administration-tools-for-windows-10-v20h2-using-configmgr-and-powershell/).

# <a name="baselines"></a>[Security Baselines](https://aka.ms/baselines)

Las líneas base de seguridad para cada versión de Windows 10 se ~~anuncian~~ anunciaban en el antiguo [Microsoft Security Guidance Blog](https://blogs.technet.microsoft.com/secguide/). Inicialmente se publicaba una versión DRAFT que, una vez revisada, se convertía en versión FINAL

{: .box-note}
**Nota**: Actualmente este contenido está archivado o migrado a [Tech Community](https://techcommunity.microsoft.com/t5/microsoft-security-baselines/bg-p/Microsoft-Security-Baselines).

| Build | Fecha de publicación |
| --- | --- |
| [Windows 10 21H1](https://techcommunity.microsoft.com/t5/microsoft-security-baselines/security-baseline-final-for-windows-10-version-21h1/ba-p/2362353) | 18 Mayo 2021 |
| [Windows 10 & Windows Server 20H2](https://techcommunity.microsoft.com/t5/microsoft-security-baselines/security-baseline-final-for-windows-10-and-windows-server/ba-p/1999393) | 17 Diciembre 2020 |
| [Windows 10 & Windows Server 2004](https://techcommunity.microsoft.com/t5/microsoft-security-baselines/security-baseline-final-windows-10-and-windows-server-version/ba-p/1543631) | 4 Agosto 2020 |
| [Windows 10 & Windows Server 1909](https://techcommunity.microsoft.com/t5/microsoft-security-baselines/security-baseline-final-for-windows-10-v1909-and-windows-server/ba-p/1023093) | 20 Noviembre 2019 |
| [Windows 10 & Windows Server 1903](https://techcommunity.microsoft.com/t5/microsoft-security-baselines/security-baseline-final-for-windows-10-v1903-and-windows-server/ba-p/701084) | 23 Mayo 2019 |
| [Windows 10 1809 / Windows Server 2019](https://techcommunity.microsoft.com/t5/microsoft-security-baselines/security-baseline-final-for-windows-10-v1809-and-windows-server/ba-p/701082) | 20 Noviembre 2018 |
| [Windows 10 1803](https://blogs.technet.microsoft.com/secguide/2018/04/30/security-baseline-for-windows-10-april-2018-update-v1803-final/) | 30 Abril 2018 |
| [Windows 10 1709](https://blogs.technet.microsoft.com/secguide/2017/10/18/security-baseline-for-windows-10-fall-creators-update-v1709-final/) | 18 Octubre 2017 |
| [Windows 10 1703](https://blogs.technet.microsoft.com/secguide/2017/08/30/security-baseline-for-windows-10-creators-update-v1703-final/) | 30 Agosto 2017 |
| [Windows 10 1607 / Windows Server 2016](https://blogs.technet.microsoft.com/secguide/2016/10/17/security-baseline-for-windows-10-v1607-anniversary-edition-and-windows-server-2016/) | 17 Octubre 2016 |
| [Windows 10 1511](https://blogs.technet.microsoft.com/secguide/2016/01/22/security-baseline-for-windows-10-v1511-threshold-2-final/) | 22 Enero 2016 |
| [Windows 10 1507](https://blogs.technet.microsoft.com/secguide/2016/01/22/security-baseline-for-windows-10-v1507-build-10240-th1-ltsb-update/) | 22 Enero 2016 |

Desde hace un tiempo, se han unificado todas las security baselines en el [**Microsoft Security Compliance Toolkit**](https://aka.ms/SCT) que incluye security baselines, herramientas y scripts para garantizar la **seguridad** de:

* Windows 10
* Windows Server
* Microsoft Office
* Microsoft Edge
* Windows Update

# <a name="adk"></a>Windows Assessment and Deployment Toolkit (Windows ADK)

Cada versión de Windows 10 requiere de unas herramientas de personalización y despliegue diferentes. Para evitar problemas, se recomienda utilizar siempre [la última versión disponible](https://docs.microsoft.com/en-us/windows-hardware/get-started/what-s-new-in-kits-and-tools).

- [1607](https://go.microsoft.com/fwlink/p/?LinkId=526740)
- [1703](https://go.microsoft.com/fwlink/p/?LinkId=845542)
- [1709](https://go.microsoft.com/fwlink/p/?linkid=859206)
- [1803](https://go.microsoft.com/fwlink/?linkid=873065)
- [1809](https://go.microsoft.com/fwlink/?linkid=2026036) + [WinPE add-on](https://go.microsoft.com/fwlink/?linkid=2022233)
- [1903](https://go.microsoft.com/fwlink/?linkid=2086042) + [WinPE add-on](https://go.microsoft.com/fwlink/?linkid=2087112)
- [2004](https://go.microsoft.com/fwlink/?linkid=2120254) + [WinPE add-on](https://go.microsoft.com/fwlink/?linkid=2120253)

# <a name="downloads">Descargas

* [Windows 10 Enterprise](https://www.microsoft.com/evalcenter/evaluate-windows-10-enterprise)
  * Evaluación por 90 días
  * 21H1 y LTSC 2019
  * 32 y 64 bits
  * Formato ISO
  * Lenguaje English, Spanish y otros
* [Windows 10 Development Environment](https://developer.microsoft.com/en-us/windows/downloads/virtual-machines/)
  * Fecha de expiración concreta
  * 20H2
  * 64 bits
  * Formato VMware, Hyper-V, VirtualBox y Parallels
  * Lenguaje English
* [IE11 and Microsoft Edge Legacy](https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/)
  * Evaluación por 90 días (se recomienda realizar snapshot antes de ejecutar)
  * 1809
  * 64 bits
  * Formato VMware, Hyper-V, VirtualBox, Parallels y Vagrant
  * Lenguaje English

# <a name="posts">Artículos
