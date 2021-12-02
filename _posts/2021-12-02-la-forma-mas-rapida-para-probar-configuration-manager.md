---
layout : post
blog-width: true
title: 'La forma más rápida para probar Configuration Manager'
date: '2021-12-02 18:27:10'
published: true
tags:
- MEMCM
author:
  display_name: Manel Rodero
---

Una de las formas más sencillas para tener un laboratorio de **Microsoft Endpoint Configuration Manager** (MEMCM) es utilizar alguno de los _Lab Kit_ que proporciona Microsoft.

![Configuration Manager][1]

Estos _Lab Kit_ son paquetes compuestos por **guías explicativas** y **máquinas virtuales** (en formato Hyper-V) que simulan un entorno real con un controlador de dominio, un servidor _ConfigMgr_, etc.

A día de hoy existen 3 kits (en realidad son 2 como veremos más adelante) que se pueden descargar desde las siguientes páginas:

* [Windows 10 and Office 365 Deployment Lab Kit](https://www.microsoft.com/en-us/evalcenter/evaluate-lab-kit)
* [Windows 11 and Office 365 Deployment Lab Kit](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-11-office-365-lab-kit)
* [Microsoft Endpoint Manager Evaluation Lab Kit](https://www.microsoft.com/en-us/evalcenter/evaluate-mem-evaluation-lab-kit)

# Windows 10 and Office 365 Deployment Lab Kit

El primer _Lab Kit_ permite desplegar una serie de máquinas virtuales que simulan un entorno real (Directorio Activo, Configuration Manager, clientes Windows 10, etc.).

Incluye unas guías en formato `*.docx` que explican detalladamente cómo se puede utilizar esta plataforma para desplegar y gestionar Windows 10 y Microsoft 365 Apps (el antiguo Office 365) en la empresa.

Los enlaces para descargar directamente este kit sin tener que registrarnos son los siguientes:

* [Win10_21H1_Labs.zip](https://download.microsoft.com/download/9/d/9/9d9e278e-a1ea-4704-85e1-cb24f3806f45/Win10_21H1_Labs.zip)
* [Win10_21H1_Guides.zip](https://download.microsoft.com/download/9/d/9/9d9e278e-a1ea-4704-85e1-cb24f3806f45/Win10_21H1_Guides.zip)

A día de hoy, este laboratorio está formado por:

- Windows 10 Enterprise, Version 21H1
- Windows 7 Enterprise Service Pack 1, Version 6.1
- Microsoft Endpoint Configuration Manager 2103 (Current Branch)
- Windows Assessment and Deployment Kit for Windows 10, Version 2004
- Microsoft Deployment Toolkit
- Microsoft BitLocker Administration and Monitoring 2.5 SP1
- Windows Server 2019
- Microsoft SQL Server 2017

Todo el laboratorio está diseñado para facilitar su uso con alguna de estas licencias:

- Microsoft 365 E5
- Office 365 Enterprise E5
- Enterprise Mobility + Security

Las **guías paso a paso** cubren las herramientas indicados en las soluciones Microsoft 365 y diversos escenarios utilizando Microsoft Endpoint Configuration Manager, Windows Analytics, Office Customization Tool, OneDrive, Windows Autopilot, etc.

# Windows 11 and Office 365 Deployment Lab Kit

Este kit es muy similar al anterior pero los clientes son Windows 11 y los servidores son Windows Server 2022.

Los enlaces para descargar directamente este kit sin tener que registrarnos son los siguientes:

* [Lab Kit_Win11_Lab.zip](https://download.microsoft.com/download/8/a/4/8a4d98e8-61a6-451f-bffc-fa9d11b178e4/Lab%20Kit_Win11_Lab.zip)
* [Lab Kit_Win11_guides.zip](https://download.microsoft.com/download/8/a/4/8a4d98e8-61a6-451f-bffc-fa9d11b178e4/Lab%20Kit_Win11_guides.zip)

# Microsoft Endpoint Manager Evaluation Lab Kit

Este _Lab Kit_ está pensado como un complemento del laboratorio de Windows 10 explicado anteriormente.

Por este motivo, incluye únicamente unas guías específicas que explican la integración con **Microsoft Intune** utilizando la consola unificada en **Microsoft Endpoint Manager** (MEM).

El enlace para descargar directamente estas guías sin tener que registrarnos es el siguiente:

* [MEM Lab_Guides.zip](https://download.microsoft.com/download/2/0/9/20915165-9072-4525-be79-1cc15e8e5c0b/MEM%20Lab_Guides.zip)

# Requerimientos

Todas las máquinas cliente y servidor de los entornos incluidos en los _Lab Kit_ son ediciones de 64-bit: Windows 10/11, Windows Server 2019/2022.

Para poder ejecutar este _Lab Kit_, es necesario disponer de un _host_ **Hyper-V** y un usuario con los permisos necesarios para crear y ejecutar máquinas virtuales.

Se recomienda disponer de 300GB de espacio libre en el disco y que éste sea lo más rápido posible (SSD o NVMe). También se recomienda 32GB de memoria RAM como mínimo y una CPU de gama alta para obtener el máximo de rendimiento.

# Instalación

Cualquiera de los dos laboratorios se instala de la misma manera:

* Descargar el fichero `*.zip`
* Extraer su contenido
* Ejecutar `setup.exe`
* Seguir las instrucciones ;-)

Este programa de instalación comprobará que el entorno cumple los requisitos mínimos para utilizar las máquinas virtuales (por ejemplo, que existe un _switch_ que proporcione acceso a Internet), descomprimirá el fichero `*.zpaq` y creará las máquinas virtuales a partir de los discos `ServerParent.vhdx` y `WindowsParent.vhdx`.

Una vez importadas, algunas máquinas se pondrán en marcha para proporcionar la infraestructura (DC1, GW1 y CM1) y otras se pondrán en marcha para aprovisionarse (CLIENT1, CLIENT2, etc.).

Durante este proceso notaremos que el uso de RAM y CPU del host Hyper-V se incrementa notablemente por lo que habrá que dejarlo un rato tranquilo (entre 30-60 minutos) hasta que se acaben de aprovisionar todas las máquinas cliente y se apaguen correctamente.

![Lab Kit en Hyper-V][2]

Al iniciar sesión en las máquinas veremos que son versiones de evaluación válidas durante determinados días en función de la fecha de descarga del kit.

A partir de aquí, únicamente queda realizar los laboratorios paso a paso siguiendo las guías de cada _Lab Kit_ ;-)

# Referencias

* [Microsoft 365 for enterprise documentation and resources](https://docs.microsoft.com/en-us/microsoft-365/enterprise/)
* [What's new in Windows 10, version 21H1 for IT Pros](https://docs.microsoft.com/en-us/windows/whats-new/whats-new-windows-10-version-21h1)
* [Deployment guide for Microsoft 365 Apps](https://docs.microsoft.com/en-us/deployoffice/deployment-guide-microsoft-365-apps)
* [Plan for Windows 10 deployment](https://docs.microsoft.com/en-us/windows/deployment/planning/)
* [Deploy updates for Windows client and Microsoft 365 apps](https://docs.microsoft.com/en-us/learn/modules/windows-deploy/) [Microsoft Learn]
* [Desktop Deployment Essentials Playlist](https://www.youtube.com/watch?v=gLUmDcCPwuM&list=PLXtHYVsvn_b_0LjDWej-d3x8C1JDEB5vh&index=2) [Microsoft Mechanics]
* [Microsoft Endpoint Manager documentation](https://docs.microsoft.com/mem)
* [Frequently asked questions for Configuration Manager branches and licensing](https://docs.microsoft.com/en-us/mem/configmgr/core/understand/product-and-licensing-faq)
* [Tutorial: Walkthrough Intune in Microsoft Endpoint Manager](https://docs.microsoft.com/en-us/mem/intune/fundamentals/tutorial-walkthrough-endpoint-manager)
* [Microsoft 365 Certified: Modern Desktop Administrator Associate](https://docs.microsoft.com/en-us/learn/certifications/m365-modern-desktop) [Microsoft Learn]

[1]: /assets/img/blog/2021-12-02_image_1.png "Configuration Manager"
[2]: /assets/img/blog/2021-12-02_image_2.png "Lab Kit en Hyper-V"
