---
layout : post
blog-width: true
title: 'Proteger las cuentas del administrador y los servicios'
date: '2020-12-11 13:10:01'
published: true
tags:
- Windows
author:
  display_name: Manel Rodero
share-img: "/assets/img/blog/2020-12-11_image_1.jpg"
---

El pasado 9 de diciembre de 2020 se celebró el evento [**MVPDays Online December 2020 - Azure Stack HCI Day**](https://www.youtube.com/watch?v=sZQN-4lykX0) moderado por [Dave Kawula](https://twitter.com/DaveKawula) y [Cristal Kawula](https://twitter.com/SuperCristal1). Aunque el evento estaba enfocado a contenido relacionado con [**Microsoft Azure Stack HCI**](https://docs.microsoft.com/en-us/azure-stack/hci/overview) y [**Storage Spaces Direct**](https://docs.microsoft.com/en-us/windows-server/storage/storage-spaces/storage-spaces-direct-overview), Mikael Nystron ha presentado contenido relacionado con la administración de la estación de trabajo y he tomado algunas notas.

## Protect your Admin Account and your Services Accounts!

[Mikael Nystron](https://twitter.com/mikael_nystrom), que trabaja en [TrueSec.com](https://www.truesec.com/) y mantiene el blog [The Deployment Bunny](https://deploymentbunny.com/), explica que la mayoría de ataques se producen al aprovechar el robo de credenciales de cuentas con privilegios de administrador y que la solución pasa por cambiar la forma de trabajar y no por utilizar un password más fuerte.

{: .box-note}
**Nota**: Al avanzar la presentación, me he dado cuenta que es muy parecida a la del webcast [Stop giving your admin credentials](https://techtalk.truesec.com/webcast/stop-giving-your-admin-credentials-away/) del pasado 26 de agosto de 2020. El vídeo está disponible en [YouTube](https://youtu.be/4hdO47xvGJc).

Si los atacantes logran entrar en un equipo en el que un administrador tiene una sesión iniciada (o la haya tenido en el pasado) pueden utilizar diferentes métodos para usar sus credenciales:

* [Pass the Hash](https://attack.stealthbits.com/pass-the-hash-attack-explained)
* [Pass the Ticket](https://attack.stealthbits.com/pass-the-ticket)

Cada que vez que se inicia sesión de forma interactiva, las credenciales se guardan en el equipo. Por tanto, sólo se deberían utilizar en dispositivos en los que confiemos. De hecho, no deberíamos confiar en ninguno: **Zero Trust** ;-)

La solución comienza por usar una [**PAW**](https://docs.microsoft.com/en-us/windows-server/identity/securing-privileged-access/privileged-access-workstations) (Privileged Access Workstation), es decir, un ordenador que:

* Ha sido desplegado y configurado por nosotros mismos
* No tiene agentes de ningún tipo, ni herramientas de monitorización
* No ha sido desplegado usando ConfigMgr, Autopilot, etc.
* Ha sido instalado directamente desde una _clean media_ 

Otros conceptos clave son:

* Proteger los privilegios administrativos: Tiering, Credential Guard, etc.
* Uso de herramientas de administración REMOTAS seguras: Windows Admin Center
* Uso de métodos de administración REMOTA seguros: PowerShell, RSAT

### Niveles

Una **cuenta privilegiada** es mucho más que la cuenta del administrador y hay que protegerlas por el daño que puede causar un robo de estas credenciales:

* Cloud service admins (Azure, AWS, GCP, etc.)
* Identity admins
* Security & management tool admins
* High impact social media accounts (Twitter, Facebook, etc.)
* Critical business data/systems

Proteger el acceso de forma segura es mucho más que proteger las contraseñas en un _vault_ y hay que proteger todo el ciclo de vida del acceso privilegiado:

* Directory
* Admin groups
* People
* Passwords/credentials
* Admin workstations
* Admin interfaces
* Etc.

Se recomienda utilizar un modelo de **3 niveles** (_tier_) para acceder a los recursos:

| Nivel | Usuarios | Desde | Hacia |
| --- | --- | --- | --- |
| Tier 0 | Forest/Domain Admins and Groups | Admin Workstations | Domain Controllers |
| Tier 1 | Server Admins and Groups | Admin Workstations | Enterprise Servers & Applications |
| Tier 2 | Workstation Admins | Admin Workstations | Workstations |

<p></p>

* Usar una cuenta _Tier n_ para iniciar sesión en un recurso de _Tier n+1_ está prohibido
* Usar una cuenta _Tier n_ para iniciar sesión en un recurso de _Tier n-1_ se hará únicamente si lo requiere el _rol_
* Los hipervisores (**The Fabric**) son _Tier -1_ porque si las VM no están protegidas con Bitlocker, TPM, etc. se tiene acceso a todo sin necesitar credenciales

Microsoft recomienda [asegurar el acceso privilegiado](https://docs.microsoft.com/en-us/windows-server/identity/securing-privileged-access/securing-privileged-access) de forma escalonada en tres fases.

* Fase 1 (primeros 30 días): Mejoras rápidas con un impacto positivo significativo
  * Cuentas separadas
  * Contraseñas de administrador local _Just in time_ usando [**LAPS**](https://stealthbits.com/blog/running-laps-in-the-race-to-security/)
  * Estaciones de trabajo administrativas
  * Detección de ataques de identidad

![PAW Fase 1][1]{:width="800px"}

* Fase 2 (90 días): Mejoras incrementales significativas
  * Requerir [**Windows Hello for Business**](https://docs.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/hello-identity-verification) y MFA
  * Implementar PAW para todos los titulares de cuentas de acceso de identidad privilegiado
  * Privilegios _just in time_
  * Habilitar [**Windows Defender Credential Guard**](https://docs.microsoft.com/en-us/windows/security/identity-protection/credential-guard/credential-guard)
  * Informes de credenciales filtradas
  * Usar Microsoft Defender para detectar _Identiy Lateral Movement Paths_

![PAW Fase 2][2]{:width="800px"}

* Fase 3 (Continuo): Mantenimiento y mejora de la seguridad
  * Revisar el control de acceso basado en roles (RBAC)
  * Reducir las superficies de ataque
  * Integrar los registros en un SIEM
  * Credenciales filtradas - Forzar restablecimiento de contraseña

![PAW Fase 3][3]{:width="800px"}

### Herramientas

Las siguientes herramientas ayudan a conseguir un entorno de administración más seguro:

* ~~Remote Desktop~~
* [Remote PowerShell](https://docs.microsoft.com/en-us/powershell/scripting/learn/remoting/running-remote-command)
* [Windows Admin Center](https://docs.microsoft.com/en-us/windows-server/manage/windows-admin-center/overview)
* [Remote Server Administration Tools](https://docs.microsoft.com/en-us/windows-server/remote/remote-server-administration-tools) (RSAT)
* [WinRM](https://docs.microsoft.com/en-us/windows/win32/winrm/portal) / WinRS
* [Server Manager](https://docs.microsoft.com/en-us/windows-server/administration/server-manager/server-manager)

{: .box-warning}
**Advertencia**: Hay que intentar no utilizar 'Remote Desktop' ya que es un agujero de seguridad si no se utiliza correctamente. Muchos ataques se basan en obtener acceso a una máquina y esperar a que un administrador inicie sesión remotamente para robarle las credenciales e intentar elevar los permisos hasta llegar al controlador de dominio.

Si no hay más remedio que utilizar 'Remote Desktop' hay un par de opciones para proteger la conexión:

* Usar el modo **Restricted Admin** mediante el comando `mstsc.exe /restrictedAdmin` para que las credenciales no se envien al equipo remoto (lo cual es bueno si este equipo ha sido comprometido)
* Usar [**Windows Defender Remote Credential Guard**](https://docs.microsoft.com/en-us/windows/security/identity-protection/remote-credential-guard), aunque no se recomienda para un entorno de _helpdesk_

Otras ideas y conceptos interesantes:

* Utilizar Smart Cards
* No usar el `Builtin\Administrator` del dominio (es una cuenta que hay que dejar para emergencias)
* Desplegar LAPS en las estaciones de trabajo y los servidores
* Las cuentas y grupos de administratores de los servidores (Tier 1) y las estaciones de trabajo (Tier 2) las gestionan los administradores de dominio (Tier 0)

## Referencias

* [Mitigating Pass-the-Hash and Other Credential Theft v1](http://download.microsoft.com/download/7/7/a/77abc5bd-8320-41af-863c-6ecfb10cb4b9/mitigating%20pass-the-hash%20(pth)%20attacks%20and%20other%20credential%20theft%20techniques_english.pdf)
* [Mitigating Pass-the-Hash and Other Credential Theft v2](http://download.microsoft.com/download/7/7/a/77abc5bd-8320-41af-863c-6ecfb10cb4b9/mitigating-pass-the-hash-attacks-and-other-credential-theft-version-2.pdf)
* [How Pass-the-Hash works](http://download.microsoft.com/download/c/3/b/c3bd2d13-fc9b-4fab-a1e7-43fc5de5cfb2/passthehashattack-datasheet.pdf)

[1]: /assets/img/blog/2020-12-11_image_1.jpg "PAW Fase 1"
[2]: /assets/img/blog/2020-12-11_image_2.jpg "PAW Fase 2"
[3]: /assets/img/blog/2020-12-11_image_3.jpg "PAW Fase 3"
