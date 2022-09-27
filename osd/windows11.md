---
layout: page
title: Windows 11
blog-width: true
---

Esta página contiene tablas y otros recursos de utilidad para los administradores del sistema operativo [Windows 11](https://docs.microsoft.com/en-us/windows/resources/){:target="_blank"}.

| [Actualizaciones](#actualizaciones) | [Ciclo de vida](#lifecycle) | [ADMX](#admx) | [RSAT](#rsat) | [Security Baselines](#baselines) | [Windows ADK](#adk) | [Descargas](#downloads) | [Artículos](#posts) |

# <a name="actualizaciones"></a>[Actualizaciones](https://aka.ms/Windows11UpdateHistory){:target="_blank"}

{: .box-note}
**Nota**: El LCU indicado es el correspondiente al _Patch Tuesday_. En Windows 11 esta actualización incluye el [SSU](https://docs.microsoft.com/en-us/windows/deployment/update/servicing-stack-updates) correspondiente.

| Versión | OS Build | Fecha actualización | Cumulative Update (CU) | Servicing Stack Update (SSU) |
| --- | --- | --- | --- |
| 21H2 (Original) | 22000.978 | 13 Septiembre 2022 | [KB5017328](https://support.microsoft.com/en-us/help/5017328){:target="_blank"} | 22000.975 |

<!-- https://en.wikipedia.org/wiki/Windows_10_version_history -->

# <a name="lifecycle"></a>[Ciclo de vida](https://support.microsoft.com/en-us/help/13853/windows-lifecycle-fact-sheet){:target="_blank"} (_lifecycle_)

{: .box-note}
**Nota**: El [final de soporte](https://docs.microsoft.com/en-us/windows/release-health/windows11-release-information) indicado es para las versiones **Enterprise** y **Education** (36 meses desde la fecha de liberación).

| Versión | Compilación | Codename | Marketing | Fecha de liberación | Final de soporte |
| --- | --- | --- | --- | --- | --- |
| 21H2 (Original) | 22000 | 21H1 | October 2021 Update | 4 Octubre 2021 | 8 Octubre 2024 |

# <a name="admx"></a>Administrative Templates (*.ADMX)

Las políticas de grupo (GPO) permiten configurar muchísimos aspectos del sistema operativo. Estas políticas de grupo se pueden crear a partir de las plantillas administrativas almacenadas localmente o en un [**almacén central**](https://docs.microsoft.com/en-us/troubleshoot/windows-client/group-policy/create-and-manage-central-store) en el **controlador de dominio**.

{: .box-note}
**Nota**: Las plantillas administrativas son acumulativas y los ficheros `*.admx` y `*.adml` (lenguajes) se pueden sobreescribir con las versiones más recientes. Estos ficheros se pueden descargar de forma manual o mediante el módulo PowerShell [EvergreenAdmx](https://github.com/msfreaks/EvergreenAdmx).

{: .box-note}
**Nota**: Para cada plantilla administrativa suele existir un fichero **Excel** con la referencia de todas las configuraciones incluidas en la misma. En algunos casos es una descarga independiente y en otros está incluida en las _security baselines_.

| Build | ADMX | Fecha de publicación | XLSX | Fecha de publicación |
| --- | --- | --- | --- | --- |
| 21H2 | [1.0](https://www.microsoft.com/en-us/download/103507) | 6 Octubre 2021 | [1.0](https://www.microsoft.com/en-us/download/103506) | 5 Octubre 2021 |

# <a name="rsat"></a>[Remote Server Administration Tool (RSAT)](https://docs.microsoft.com/en-us/troubleshoot/windows-server/system-management-components/remote-server-administration-tools)

Las herramientas de administración remota del servidor para Windows 10 1803 o anteriores se tienen que [descargar desde Microsoft](https://www.microsoft.com/en-us/download/details.aspx?id=45520). A partir de Windows 10 1809 se instalan como una **Feature on Demand**.

{: .box-note}
**Nota**: Se ha documentado el proceso en el post "[Añadir RSAT en Windows 10 1809](/blog/anadir-rsat-en-windows-10-1809)" publicado en el blog. Existe un [script en PowerShell](https://github.com/imabdk/Powershell/blob/master/Install-RSATv1809v1903v1909v2004v20H2.ps1) creado por [Martin Bengtsson](https://www.imab.dk/) que se puede usar para instalar más fácilmente, incluso desde [Software Center](https://www.imab.dk/deploy-rsat-remote-server-administration-tools-for-windows-10-v20h2-using-configmgr-and-powershell/).

# <a name="baselines"></a>[Security Baselines](https://aka.ms/baselines)

Desde hace un tiempo, se han unificado todas las security baselines en el [**Microsoft Security Compliance Toolkit**](https://aka.ms/SCT) que incluye security baselines, herramientas y scripts para garantizar la **seguridad** de:

* Windows 11
* Windows 10
* Windows Server
* Microsoft 365 Apps (Office)
* Microsoft Edge

| Versión | Fecha de publicación |
| --- | --- |
| [1.0](https://www.microsoft.com/en-us/download/details.aspx?id=55319) | 13 Septiembre 2022 |

# <a name="adk"></a>Windows Assessment and Deployment Toolkit (Windows ADK)

Cada versión de Windows 11 requiere de unas herramientas de personalización y despliegue diferentes. Para evitar problemas, se recomienda utilizar siempre [la última versión disponible](https://docs.microsoft.com/en-us/windows-hardware/get-started/what-s-new-in-kits-and-tools).

| ADK | Notas |
| --- | --- |
| [22H2](https://go.microsoft.com/fwlink/?linkid=2196127) + [WinPE add-on](https://go.microsoft.com/fwlink/?linkid=2196224) | [ConfigMgr 2111+](https://www.deploymentresearch.com/notes-from-the-lab-on-windows-adk-for-windows-11-22h2/) |
| [21H2](https://go.microsoft.com/fwlink/?linkid=2165884) + [WinPE add-on](https://go.microsoft.com/fwlink/?linkid=2166133) | |

# <a name="downloads">Descargas

* [Windows 11 ISO](https://www.microsoft.com/en-us/software-download/windows11)
  * Licencia requerida
  * 22H2
  * 64 bits
  * Opciones:
    * Windows 11 Installation Assistant
    * Create Windows 11 Installation Media
    * Windows 11 Disk Image (ISO)
* [Windows 11 Enterprise](https://www.microsoft.com/evalcenter/evaluate-windows-11-enterprise)
  * Evaluación por 90 días
  * 22H2
  * 64 bits
  * Formato ISO
  * Lenguaje English, Spanish y otros
* [Windows 11 Development Environment](https://developer.microsoft.com/en-us/windows/downloads/virtual-machines/)
  * Fecha de expiración concreta
  * 21H1
  * 64 bits
  * Formato VMware, Hyper-V, VirtualBox y Parallels
  * Lenguaje English

# <a name="posts">Artículos

| Fecha | Artículo | Autor | Notas |
| --- | --- | --- | --- |
| 22 Octubre 2021 | [Configure Windows 11 Start Menu folders using PowerShell](https://blog.onevinn.com/configure-windows-11-start-menu-folders-using-powershell){:target="_blank"} | [Jörgen Nilsson](https://twitter.com/ccmexec) | Personalización de las carpetas que aparecen en el Inicio junto al botón de encendido (p.ej. se puede añadir un acceso más rápido a la Configuración) |
| Octubre 2021 | [Customizing Windows 11 default Start Menu during OSD using LayoutModification.json](https://blog.onevinn.com/customizing-windows-11-default-start-menu-during-osd-using-layoutmodification-json) | [Jörgen Nilsson](https://twitter.com/ccmexec) | |
| 19 Octubre 2021 | [Modify Windows 11 Taskbar during OSD, Intune and GPO](https://blog.onevinn.com/modify-windows-11-taskbar-during-osd-intune-and-gpo) | [Jörgen Nilsson](https://twitter.com/ccmexec) | |
| 18 Octubre 2021 | [Modifying Windows 11 Start button location and Taskbar icons during OSD/AutoPilot](https://blog.onevinn.com/modifying-windows-11-start-button-location-and-taskbar-icons-during-osd-autopilot) | [Jörgen Nilsson](https://twitter.com/ccmexec) | |
| 17 Octubre 2021 | [Remove the chat icon in Windows 11 Start menu using GPO/Intune](https://blog.onevinn.com/remove-the-chat-icon-in-windows-11-start-menu-using-gpo-intune) | [Jörgen Nilsson](https://twitter.com/ccmexec) | |
| 6 Octubre 2021 | [Windows 11 customizations a first look](https://blog.onevinn.com/windows-11-customizations-a-first-look) | [Jörgen Nilsson](https://twitter.com/ccmexec) | Algunos scripts para personalizar Windows 11 (p.ej. Branding de la pantalla de inicio o las asociaciones de las aplicaciones) |