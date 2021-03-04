---
layout : post
blog-width: true
title: 'Creación de un "Dev Tenant" de Microsoft 365'
date: '2021-03-04 21:17:01'
published: true
tags:
- Microsoft 365
author:
  display_name: Manel Rodero
---

Hace unos días, una MVP llamada [Julie Turner](https://twitter.com/jfj1997) publicaba el artículo [What is a "Dev Tenant" and why would you want one?](https://techcommunity.microsoft.com/t5/microsoft-365-pnp-blog/what-is-a-dev-tenant-and-why-would-you-want-one/ba-p/2036610) en la [Tech Community](https://techcommunity.microsoft.com/) de Microsoft.

Me ha parecido muy interesante la idea de poder disponer de una [subscripción para desarrolladores de Microsoft 365 E5](https://developer.microsoft.com/en-us/microsoft-365/dev-program#Subscription) para probar esta plataforma y aprender poco a poco sin tener que estar pendiente de una fecha de caducidad del _tenant_.

{: .box-note}
**Nota**: Es necesario utilizar el _tenant_ para que se vaya renovando cada 90 días.

Las características principales de esta suscripción son las siguientes:

* Incluye 25 licencias con fines de desarrollo
* Acceso a los _workloads_ y capacidades principales de Microsoft 365:
  * Todas las aplicaciones Office 365 (incluyendo SharePoint, OneDrive, Outlook/Exchange, Teams, Planner, Word, Excel, PowerPoint y muchas más)
  * Office 365 Advanced Threat Protection
  * Analíticas avanzadas con Power BI
  * Enterprise Mobility + Security (EMS) para el cumplimiento y la protección de la información
  * Azure Active Directory para crear soluciones avanzadas de gestión de identidades y accesos

{: .box-note}
**Nota**: No incluye Windows 10, pero se pueden utilizar [versiones de evaluación](https://www.microsoft.com/es-es/evalcenter/evaluate-windows-10-enterprise) para probar cualquier escenario.

Los créditos _complimentary_ de Azure [se pueden utilizar en este entorno de desarrollo](https://laurakokkarinen.com/how-to-use-the-complimentary-azure-credits-in-a-microsoft-365-developer-tenant-step-by-step/) para poder simular cualquier entorno de producción real.

Además, es posible cargar datos de ejemplo (usuarios, eventos de correo, _sites_ de SharePoint, etc.) desde el [Dashboard - Microsoft 365 Dev Center](https://developer.microsoft.com/en-us/microsoft-365/profile) para facilitar las pruebas.

## Unirse al programa

Los pasos para unirse al programa y crear este _tenant_ para practicar y aprender son los siguientes:

1) Acceder a la página principal del [Developer Program - Microsoft 365](https://developer.microsoft.com/en-us/microsoft-365/dev-program)

2) Pulsar el botón [`Join now`](https://developer.microsoft.com/microsoft-365/profile)

3) Autenticarse mediante una cuenta Microsoft

4) Rellenar el formulario y pulsar el botón `Next`:

```
Country/Region: Spain
Company: <>
Language preference: English
[x] I accept the terms and conditions ...
[x] I would like information, tips, ...
Primary focus as developer?
  ( ) Applications to be sold in market
  ( ) Custom solutions for my own customers
  (*) Applications for internal use at my company
  ( ) Personal projects
Areas of Microsoft 365 development?
  [ ] SharePoint Framework (SPFx)
  [x] Microsoft Graph
  [x] Microsoft Teams
  [ ] Office Add-ins
  [ ] Outlook
  [ ] Microsoft identity platform
  [x] Power Platform
```

5) Aceptar el mensaje de bienvenida:

```
Welcome to the Microsoft 365 Developer Program!
If you don't have a subscription already, we recommend that you begin your membership by setting up your new Microsoft 365 developer subscription.
```

6) Pulsar el botón `Set up E5 subscription` y rellenar el formulario:

```
Country/Region: Spain
Create username: <>
Create domain: <>
Password: <>
Country code: Spain (+34)
Phone number: <>
```

7) Introducir el código que llegará por SMS y esperar unos minutos a que el _tenant_ se aprovisione

Una vez creado es necesario [asignarse una licencia](https://docs.microsoft.com/en-us/office365/admin/subscriptions-and-billing/assign-licenses-to-users?view=o365-worldwide) para acceder a todas las funcionalidades. En [Microsoft Learn for Microsoft 365](https://support.office.com/en-us/article/license-and-user-location-management-f1ae424a-5554-4376-8cad-778bbc32063d) explica como hacerlo aunque es bastante sencillo.

{: .box-note}
**Nota**: Uno de los primeros pasos para poder explorar y aprender sería agregar el [_sample data pack_](https://docs.microsoft.com/en-us/office/developer-program/install-sample-packs) de usuarios. Se dispone así de 16 usuarios ficticios con licencias, buzones, nombres y fotos de cada uno, etc. para simular un entorno real.

A partir de aquí se puede gestionar las características del programa desde el [Microsoft 365 Developer Program Dashboard](https://developer.microsoft.com/en-us/microsoft-365/profile/).

![M365 Developer Subscription][1]

Ahora sólo queda "jugar" para aprender y quizas obtener alguna [certificación de Microsoft](http://aka.ms/TrainCertPoster) como por ejemplo:

* [MS-900: Microsoft 365 Fundamentals](https://docs.microsoft.com/en-us/learn/certifications/exams/ms-900)
* [MD-100: Windows 10](https://docs.microsoft.com/en-us/learn/certifications/exams/md-100)
* [MD-101: Managing Modern Desktops](https://docs.microsoft.com/en-us/learn/certifications/exams/md-101)
* etc.

## Referencias

* [Microsoft 365 Licensing](https://m365maps.com/), diagramas de licencia de [Aaron Dinnage](https://twitter.com/AaronDinnage)
* [Intune Training](https://www.youtube.com/channel/UCfmMlhX5TW8cicxHw6ExYVA), canal de YouTube sobre Intune de [Adam Gross](https://www.twitter.com/AdamGrossTX), [Steve Hosking](https://www.twitter.com/OnPremCloudGuy) y [Ben Reader](https://twitter.com/powers_hell)
* [S02E17 - Microsoft Intune and Autopilot Quick Start Guide (2020 Edition)](https://www.youtube.com/watch?v=OYaDWKqg1uY), vídeo del canal anterior pensado como introducción
* [Microsoft Endpoint Manager](https://www.youtube.com/playlist?list=PLXPr7gfUMmKyf3-7Acn59TbU1grX9V2_X), playlist del canal de YouTube de [Microsoft 365](https://www.youtube.com/c/microsoft365)
* [Windows and Office deployment lab kit](https://docs.microsoft.com/en-us/microsoft-365/enterprise/modern-desktop-deployment-and-management-lab)

<p></p>

[1]: /assets/img/blog/2021-03-04_image_1.png "M365 Developer Subscription"
