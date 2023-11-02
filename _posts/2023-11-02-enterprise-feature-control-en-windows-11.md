---
layout : post
blog-width: true
title: 'Enterprise Feature Control en Windows 11'
date: '2023-11-02 17:49:22'
published: true
tags:
- Windows
author:
  display_name: Manel Rodero
---

Hace un par de días, el 31 de Octubre de 2023, se anunció el lanzamiento oficial de [Windows 11 2023 Update](https://blogs.windows.com/windowsexperience/2023/10/31/how-to-get-the-windows-11-2023-update/) también conocido como **Windows 11 23H2**.

El 26 de Septiembre de 2023 se anunciaron las [nuevas características de Windows 11](https://blogs.windows.com/windowsexperience/2023/09/26/the-most-personal-windows-11-experience-begins-rolling-out-today/), por ejemplo:

* Copilot en Windows
* Herramientas AI en Paint
* Edición de vídeo con Microsoft Clipchamp
* Extracción de texto en la Snipping Tool (herramienta de recorte)
* etc.

Cómo quería echarles un vistazo, activé la opción "[Get Windows updates as soon as they're available](https://support.microsoft.com/en-us/windows/get-windows-updates-as-soon-as-they-re-available-for-your-device-cad7b32b-001e-435b-9110-f18309b54168)" (Obtenga actualizaciones de Windows tan pronto como estén disponibles) y procedí a comprobar las actualizaciones:

![][1]

Para mi sorpresa, no se instaló la versión 23H2, pero sí una opción que me ha llamado la atención, **Configuration Update (KB5030509)**:

![][2]

# Configuration Update

Si se accede al histórico de actualizaciones, hay un enlace que te permite obtener más información y se llega al artículo [_September 23, 2023 - Windows Configuration Update_](https://support.microsoft.com/en-us/topic/september-26-2023-windows-configuration-update-542780c2-594c-46cb-979d-11116fe164ba) donde se indica que se trata de un componente que permite la [**entrega contínua de innovación en Windows 11**](https://support.microsoft.com/en-us/windows/delivering-continuous-innovation-in-windows-11-b0aa0a27-ea9a-4365-9224-cb155e517f12).

![][3]

Esta entrega contínua permite probar antes características que estarán disponibles en la siguiente **Feature Update** (que aparece en el segundo semestre del año, es decir, `xxH2`). Para ello utiliza lo que Microsoft llama _Controlled Feature Rollout (CFR)_ en la que van liberando características poco a poco. A medida que su telemetría les indica que la característica está lista, se despliega a más dispositivos y, finalmente, se añade a un **Cumulative Update** en los meses siguientes.

Toda la información obtenida por la telemetría, información de versiones y problemas, etc. se puede ver en un _dashboard_ llamado [**Windows Release Health**](https://learn.microsoft.com/en-us/windows/release-health/).

En un entorno empresarial se puede controlar la entrega de estas características mediante la GPO `Enable features introduced via servicing that are off by default` o el CSP `AllowTemporaryEnterpriseFeatureControl` que Aria Carley ([@ariaupdated](https://twitter.com/ariaupdated)) explica detalladamente en el artículo "[Commercial control for continuous innovation](https://techcommunity.microsoft.com/t5/windows-it-pro-blog/commercial-control-for-continuous-innovation/ba-p/3737575)" del 9 de Febrero de 2023.

En el KB de cada actualización mensual se describen las características que están desactivas por defecto y los administradores podemos activarlas si son necesarias usando los [**Enterprise Feature Control**](https://learn.microsoft.com/en-US/windows/whats-new/temporary-enterprise-feature-control) de Windows 11.

# ¿Al final se instaló la 23H2?

Después de haberse instalado las actualizaciones que he comentado al principio del artículo, mi Windows 11 estaba en la versión `22621.2506` que se corresponde con el [KB5031455](https://support.microsoft.com/en-us/topic/october-31-2023-kb5031455-os-builds-22621-2506-and-22631-2506-preview-6513c5ec-c5a2-4aaf-97f5-44c13d29e0d4):

![][4]

Se trata de una actualización acumulativa (LCU) que incluye características como la versión preliminar de **Copilot in Windows** tal como se explica en el artículo del [KB5031455](https://support.microsoft.com/en-us/topic/october-31-2023-kb5031455-os-builds-22621-2506-and-22631-2506-preview-6513c5ec-c5a2-4aaf-97f5-44c13d29e0d4).

Ya, pero ¿se instaló la 23H2? De momento no, pero ese KB5031455 es un requisito para poder instalar el [**KB5027397**](https://support.microsoft.com/en-us/topic/kb5027397-feature-update-to-windows-11-version-23h2-by-using-an-enablement-package-b9e76726-3c94-40de-b40b-99decba3db9d), que corresponde al _enablement package_ de la **Feature update to Windows 11, version 23H2**.

Por tanto, es cuestión tiempo que se acabe instalando ;-)

# Referencias

* [What's new in Windows 11, version 23H2](https://learn.microsoft.com/en-us/windows/whats-new/whats-new-windows-11-version-23h2)
* [What’s new for IT pros in Windows 11, version 23H2](https://techcommunity.microsoft.com/t5/windows-it-pro-blog/what-s-new-for-it-pros-in-windows-11-version-23h2/ba-p/3967814)

[1]: /assets/img/blog/2023-11-02_image_1.png "Fast Updates"
[2]: /assets/img/blog/2023-11-02_image_2.png "Configuration Update"
[3]: /assets/img/blog/2023-11-02_image_3.png "Learn More KB5030509"
[4]: /assets/img/blog/2023-11-02_image_4.png "CU Preview KB5031455"