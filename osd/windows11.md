---
layout: page
title: Windows 11
blog-width: true
---

Esta página contiene tablas y otros recursos de utilidad para los administradores del sistema operativo [Windows 11](https://docs.microsoft.com/en-us/windows/resources/){:target="_blank"}.

| [Actualizaciones](#actualizaciones) | [Ciclo de vida](#lifecycle) | [ADMX](#admx) | [RSAT](#rsat) | [Security Baselines](#baselines) |
| [Windows ADK](#adk) | [Aplicaciones universales](#appx) | [Descargas](#downloads) | [Artículos](#posts) |

# <a name="actualizaciones"></a>[Actualizaciones](https://aka.ms/Windows11UpdateHistory){:target="_blank"}

{: .box-note}
**Nota**: El LCU indicado es el correspondiente al _Patch Tuesday_. En Windows 11 esta actualización incluye el [SSU](https://docs.microsoft.com/en-us/windows/deployment/update/servicing-stack-updates){:target="_blank"} correspondiente.

| Versión | OS Build | Fecha actualización | Cumulative Update (CU) | Servicing Stack Update (SSU) |
| --- | --- | --- | --- |
| 22H2 | 22621.2428 | 10 Octubre 2023 | [KB5031354](https://support.microsoft.com/en-us/help/5031354){:target="_blank"} | 22621.2423 |
| 21H2 (Original) | 22000.2538 | 10 Octubre 2023 | [KB5031358](https://support.microsoft.com/en-us/help/5031358){:target="_blank"} | 22000.2531 |

<!-- https://en.wikipedia.org/wiki/Windows_10_version_history -->

# <a name="lifecycle"></a>[Ciclo de vida](https://support.microsoft.com/en-us/help/13853/windows-lifecycle-fact-sheet){:target="_blank"} (_lifecycle_)

{: .box-note}
**Nota**: El [final de soporte](https://docs.microsoft.com/en-us/windows/release-health/windows11-release-information){:target="_blank"} indicado es para las versiones **Enterprise** y **Education** (36 meses desde la fecha de liberación).

| Versión | Compilación | Codename | Marketing | Fecha de liberación | Final de soporte |
| --- | --- | --- | --- | --- | --- |
| 22H2 | 22621 | 22H2 | September 2022 Update | 20 Septiembre 2022 | 14 Octubre 2025 |
| 21H2 (Original) | 22000 | 21H2 | October 2021 Update | 4 Octubre 2021 | 8 Octubre 2024 |

# <a name="admx"></a>Administrative Templates (*.ADMX)

Las políticas de grupo (GPO) permiten configurar muchísimos aspectos del sistema operativo. Estas políticas de grupo se pueden crear a partir de las plantillas administrativas almacenadas localmente o en un [**almacén central**](https://docs.microsoft.com/en-us/troubleshoot/windows-client/group-policy/create-and-manage-central-store){:target="_blank"} en el **controlador de dominio**.

{: .box-note}
**Nota**: Las plantillas administrativas son acumulativas y los ficheros `*.admx` y `*.adml` (lenguajes) se pueden sobreescribir con las versiones más recientes. Estos ficheros se pueden descargar de forma manual o mediante el módulo PowerShell [EvergreenAdmx](https://github.com/msfreaks/EvergreenAdmx){:target="_blank"}.

{: .box-note}
**Nota**: Para cada plantilla administrativa suele existir un fichero **Excel** con la referencia de todas las configuraciones incluidas en la misma. En algunos casos es una descarga independiente y en otros está incluida en las _security baselines_.

{: .box-note}
**Nota**: Desde el 21 Julio 2023 se pueden usar las plantillas de Windows 11 para administrar Windows 10. Más información en el artículo "[Windows 10 or Windows 11 GPO ADMX – An Update](https://techcommunity.microsoft.com/t5/core-infrastructure-and-security/windows-10-or-windows-11-gpo-admx-an-update/ba-p/3703548)".

| Build | ADMX | Fecha de publicación | XLSX | Fecha de publicación |
| --- | --- | --- | --- | --- |
| 22H2 | [3.0](https://www.microsoft.com/en-us/download/105390){:target="_blank"} | 21 Julio 2023 | [2.0](https://www.microsoft.com/en-us/download/105094){:target="_blank"} | 15 Marzo 2023 |
| 21H2 | [1.0](https://www.microsoft.com/en-us/download/103507){:target="_blank"} | 6 Octubre 2021 | [1.0](https://www.microsoft.com/en-us/download/103506){:target="_blank"} | 5 Octubre 2021 |

# <a name="rsat"></a>[Remote Server Administration Tool (RSAT)](https://docs.microsoft.com/en-us/troubleshoot/windows-server/system-management-components/remote-server-administration-tools){:target="_blank"}

Las herramientas de administración remota del servidor para Windows 10 1803 o anteriores se tienen que [descargar desde Microsoft](https://www.microsoft.com/en-us/download/details.aspx?id=45520){:target="_blank"}. A partir de Windows 10 1809 se instalan como una **Feature on Demand**.

{: .box-note}
**Nota**: Se ha documentado el proceso en el post "[Añadir RSAT en Windows 10 1809](/blog/anadir-rsat-en-windows-10-1809){:target="_blank"}" publicado en el blog. Existe un [script en PowerShell](https://github.com/imabdk/Powershell/blob/master/Install-RSATv1809v1903v1909v2004v20H2.ps1){:target="_blank"} creado por [Martin Bengtsson](https://www.imab.dk/){:target="_blank"} que se puede usar para instalar más fácilmente, incluso desde [Software Center](https://www.imab.dk/deploy-rsat-remote-server-administration-tools-for-windows-10-v20h2-using-configmgr-and-powershell/){:target="_blank"}.

# <a name="baselines"></a>[Security Baselines](https://aka.ms/baselines){:target="_blank"}

Desde hace un tiempo, se han unificado todas las security baselines en el [**Microsoft Security Compliance Toolkit**](https://aka.ms/SCT){:target="_blank"} que incluye security baselines, herramientas y scripts para garantizar la **seguridad** de:

* Windows 11
* Windows 10
* Windows Server
* Microsoft 365 Apps (Office)
* Microsoft Edge

| Versión | Fecha de publicación |
| --- | --- |
| [1.0](https://www.microsoft.com/en-us/download/details.aspx?id=55319){:target="_blank"} | 11 Octubre 2023 |

# <a name="adk"></a>Windows Assessment and Deployment Toolkit (Windows ADK)

Cada versión de Windows 11 requiere de unas herramientas de personalización y despliegue diferentes. Para evitar problemas, se recomienda utilizar siempre [la última versión disponible](https://docs.microsoft.com/en-us/windows-hardware/get-started/what-s-new-in-kits-and-tools){:target="_blank"}.

| ADK | Build | Notas |
| --- | --- | --- |
| [22H2](https://go.microsoft.com/fwlink/?linkid=2243390){:target="_blank"} + [WinPE add-on](https://go.microsoft.com/fwlink/?linkid=2243391){:target="_blank"} | 10.1.22621.1 (Septiembre 2023) | [ConfigMgr 2207+](https://learn.microsoft.com/en-us/mem/configmgr/core/plan-design/configs/support-for-windows-adk){:target="_blank"} |
| [22H2](https://go.microsoft.com/fwlink/?linkid=2196127){:target="_blank"} + [WinPE add-on](https://go.microsoft.com/fwlink/?linkid=2196224){:target="_blank"} | 10.1.22000 | [ConfigMgr 2111+](https://www.deploymentresearch.com/notes-from-the-lab-on-windows-adk-for-windows-11-22h2/){:target="_blank"} |
| [21H2](https://go.microsoft.com/fwlink/?linkid=2165884){:target="_blank"} + [WinPE add-on](https://go.microsoft.com/fwlink/?linkid=2166133){:target="_blank"} | 10.1.19041 | |

# <a name="appx"></a>Aplicaciones universales

Listado de aplicaciones universales (Appx) incluídas con cada versión del sistema operativo, añadiendo la decisión de mantenerlas o eliminarlas durante la construcción de una [OSBuild](https://github.com/manelrodero/OSDBuilder#eliminar-aplicaciones-universales-appx){:target="_blank"}:

| Paquete | 21H2 | 22H2 | Aplicación | Decisión |
| --- | :---: | :---: | --- | :---: |
| Clipchamp.Clipchamp | | x | Editor de vídeo | **Eliminar** |
| Microsoft.549981C3F5F10 | x | x | Cortana | Conservar |
| Microsoft.BingNews | x | x | Noticias | **Eliminar** |
| Microsoft.BingWeather | x | x | El Tiempo | **Eliminar** |
| Microsoft.DesktopAppInstaller | x | x | Instalador de aplicación | Conservar |
| Microsoft.GamingApp | x | x | Servicios de juegos | **Eliminar** |
| Microsoft.GetHelp | x | x | Obtener ayuda | **Eliminar** |
| Microsoft.Getstarted | x | x | Recomendaciones | **Eliminar** |
| Microsoft.HEIFImageExtension | x | x | Extensiones de imagen HEIF | Conservar |
| Microsoft.HEVCVideoExtension | | x | Extensiones de vídeo HEVC | Conservar |
| Microsoft.MicrosoftOfficeHub | x | x | Office | **Eliminar** |
| Microsoft.MicrosoftSolitaireCollection | x | x | Microsoft Solitaire Collection | **Eliminar** |
| Microsoft.MicrosoftStickyNotes | x | x | Sticky Notes | Conservar |
| Microsoft.Paint | x | x | Paint | Conservar |
| Microsoft.People | x | x | Contactos | **Eliminar** |
| Microsoft.PowerAutomateDesktop | x | x | Power Automate | Conservar |
| Microsoft.RawImageExtension | | x | Extensiones de imagen Raw | Conservar |
| Microsoft.ScreenSketch | x | x | Recorte y anotación | Conservar |
| Microsoft.SecHealthUI | x | x | | Conservar |
| Microsoft.StorePurchaseApp | x | x | Store Purchase App | Conservar |
| Microsoft.Todos | x | x | Aplicación To-Do | Conservar |
| Microsoft.UI.Xaml.2.4 | x | | | Conservar |
| Microsoft.VCLibs.140.00 | x | x | C++ Runtime for Desktop Bridge | Conservar |
| Microsoft.VP9VideoExtensions | x | x | VP9 Video Extensions | Conservar |
| Microsoft.WebMediaExtensions | x | x | Extensiones de multimedia web | Conservar |
| Microsoft.WebpImageExtension | x | x | Extensiones de imagen Webp | Conservar |
| Microsoft.Windows.Photos | x | x | Fotos | Conservar |
| Microsoft.WindowsAlarms | x | x | Alarmas y reloj | Conservar |
| Microsoft.WindowsCalculator | x | x | Calculadora | Conservar |
| Microsoft.WindowsCamera | x | x | Cámara | Conservar |
| microsoft.windowscommunicationsapps | x | x | Correo y Calendario | **Eliminar** |
| Microsoft.WindowsFeedbackHub | x | x | Centro de opiniones | Conservar |
| Microsoft.WindowsMaps | x | x | Mapas | **Eliminar** |
| Microsoft.WindowsNotepad | x | x | Bloc de notas | Conservar |
| Microsoft.WindowsSoundRecorder | x | x | Grabadora de voz | Conservar |
| Microsoft.WindowsStore | x | x | Microsoft Store | Conservar |
| Microsoft.WindowsTerminal | x | x | Terminal | Conservar |
| Microsoft.Xbox.TCUI | x | x | Experiencia de Xbox Live en el juego | **Eliminar** |
| Microsoft.XboxGameOverlay | x | x | Complemento de la barra de juego Xbox | **Eliminar** |
| Microsoft.XboxGamingOverlay | x | x | Barra de juego de Xbox | **Eliminar** |
| Microsoft.XboxIdentityProvider | x | x | Proveedor de identidades de Xbox | **Eliminar** |
| Microsoft.XboxSpeechToTextOverlay | x | x | | **Eliminar** |
| Microsoft.YourPhone | x | x | Tu Teléfono | **Eliminar** |
| Microsoft.ZuneMusic | x | x | Groove Música | **Eliminar** |
| Microsoft.ZuneVideo | x | x |  Películas y TV | **Eliminar** |
| MicrosoftCorporationII.QuickAssist | | x | Asistencia rápida | Conservar |
| MicrosoftWindows.Client.WebExperience | x | x | | **Eliminar** |

# <a name="downloads">Descargas

* [Windows 11 ISO](https://www.microsoft.com/en-us/software-download/windows11){:target="_blank"}
  * Licencia requerida
  * 22H2
  * 64 bits
  * Opciones:
    * Windows 11 Installation Assistant
    * Create Windows 11 Installation Media
    * Windows 11 Disk Image (ISO) for x64 devices
* [Windows 11 Enterprise](https://www.microsoft.com/evalcenter/evaluate-windows-11-enterprise){:target="_blank"}
  * Evaluación por 90 días
  * 22H2
  * 64 bits
  * Formato ISO
  * Lenguaje English, Spanish y otros
* [Windows 11 Development Environment](https://developer.microsoft.com/en-us/windows/downloads/virtual-machines/){:target="_blank"}
  * Fecha de expiración concreta
  * 64 bits
  * Formato VMware, Hyper-V, VirtualBox y Parallels
  * Lenguaje English
* [Windows 11 Insider](https://learn.microsoft.com/en-us/windows-insider/ISOs)
  * Formato ISO

# <a name="posts">Artículos

| Fecha | Artículo | Autor | Notas |
| --- | --- | --- | --- |
| 22 Octubre 2021 | [Configure Windows 11 Start Menu folders using PowerShell](https://blog.onevinn.com/configure-windows-11-start-menu-folders-using-powershell){:target="_blank"} | [Jörgen Nilsson](https://twitter.com/ccmexec){:target="_blank"} | Personalización de las carpetas que aparecen en el Inicio junto al botón de encendido (p.ej. se puede añadir un acceso más rápido a la Configuración) |
| Octubre 2021 | [Customizing Windows 11 default Start Menu during OSD using LayoutModification.json](https://blog.onevinn.com/customizing-windows-11-default-start-menu-during-osd-using-layoutmodification-json){:target="_blank"} | [Jörgen Nilsson](https://twitter.com/ccmexec){:target="_blank"} | |
| 19 Octubre 2021 | [Modify Windows 11 Taskbar during OSD, Intune and GPO](https://blog.onevinn.com/modify-windows-11-taskbar-during-osd-intune-and-gpo){:target="_blank"} | [Jörgen Nilsson](https://twitter.com/ccmexec){:target="_blank"} | |
| 18 Octubre 2021 | [Modifying Windows 11 Start button location and Taskbar icons during OSD/AutoPilot](https://blog.onevinn.com/modifying-windows-11-start-button-location-and-taskbar-icons-during-osd-autopilot){:target="_blank"} | [Jörgen Nilsson](https://twitter.com/ccmexec){:target="_blank"} | |
| 17 Octubre 2021 | [Remove the chat icon in Windows 11 Start menu using GPO/Intune](https://blog.onevinn.com/remove-the-chat-icon-in-windows-11-start-menu-using-gpo-intune){:target="_blank"} | [Jörgen Nilsson](https://twitter.com/ccmexec){:target="_blank"} | |
| 6 Octubre 2021 | [Windows 11 customizations a first look](https://blog.onevinn.com/windows-11-customizations-a-first-look){:target="_blank"} | [Jörgen Nilsson](https://twitter.com/ccmexec){:target="_blank"} | Algunos scripts para personalizar Windows 11 (p.ej. Branding de la pantalla de inicio o las asociaciones de las aplicaciones) |