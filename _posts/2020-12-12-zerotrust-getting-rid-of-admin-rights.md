---
layout : post
blog-width: true
title: 'ZeroTrust – Getting rid of Admin Rights'
date: '2020-12-12 19:16:09'
published: true
tags:
- Windows
- Security
- ZeroTrust
author:
  display_name: Manel Rodero
cover-img: [ "/assets/img/blog/2020-12-12_cover.jpg" : "Foto: 42gears.com" ]
thumbnail-img: ""
---

El pasado 4 de diciembre de 2020 asistí a un taller impartido por [Sami Laiho](https://twitter.com/samilaiho) titulado "**ZeroTrust – Getting rid of Admin Rights**" en el que se explicó cómo deshacerse de los derechos de administrador local de los usuarios finales en esta nueva era de [**Seguridad de Confianza Cero**](https://docs.microsoft.com/en-us/security/zero-trust/).

![Fuente: Microsoft][1]

* Sami dice: "_Quitar los derechos de administrador del usuario final puede reducir la cantidad de tickets de Helpdesk en un 75%_"
* La mayoría de la gente dice: "_Si no tengo derechos de administrador, no puedo arreglar mi ordenador_"
* Sami contesta: "**_En realidad, ¡si no tienes derechos de administrador, no puedes dañar tu ordenador!_**"

## La aproximación de Sami

* Tener un inventario hardware y software actualizado
* Bitlocker (encriptación)
* Principio del menor privilegio posible para hacer el trabajo (Least Privilege)
* Modelo de niveles AD (Tier Model): un administrador de dominio no puede iniciar sesión en una estación de trabajo
* Uso de estaciones de trabajo administrativas (PAW): RDP es para emergencias; se necesita una estación de trabajo administrativa
* Introducción de IPsec
* Utilizar ~~_whitelisting_~~ _allowlisting_ cuando sea posible
* MFA, autenticación fuerte

## Zero Trust

A diferencia de épocas anteriores donde estuvo de moda el BYOD, ahora la aproximación _Zero Trust_ intenta:

* Empoderar a los usuarios para trabajar de forma más segura en cualquier sitio, a cualquier hora y en cualquier dispostivo
* Permitir la trasnformación digital con una inteligencia de seguridad en los entornos complejos de hoy
* Cerrar las brechas de seguridad y minimizar el riesgo de movimientos laterales

Los principios de la Confianza Cero son:

* Verificar explícitamente (qué usuario és)
* Usar el acceso con menos privilegios (nunca con permisos de administrador)
* Asumir que puede haber brechas

{: .box-note}
**Nota**: En lugar de asumir que cualquier cosa detrás del cortafuegos corporativo es seguro, el modelo **Zero Trust** asume que puede haber brechas y verifica cada petición como si ésta se originara desde una red abierta. **Never trust, always verify**.

Los componentes principales de _Zero Trust_ son:

* **Identidades**: Verificar y securizar cada identidad con autenticación fuerte
* **Dispositivos**: Obtener visibilidad de los dispositivos que acceden a la red. Asegurar el cumplimiento y el estado de salud antes de permitir el acceso
* **Aplicaciones**: Descubrir si existe _shadow IT_, asegurar los permisos en las aplicaciones y monitorizar y controlar las acciones de los usuarios
* **Datos**: Move from perimeter-based data protection to data-driven protection
* **Infraestructura**: Usar telemetría para detectar ataques y usar los principios del menor privilegio posible
* **Red**: Encriptar todas las comunicaciones internas, limitar el acceso por políticas

### Zero Trust: Least Privilege

![Grupo Administradores][2]

{: .box-warning}
**Atención**: La Guía del Usuario de Windows (desde la época de [Windows NT 3.1](https://en.wikipedia.org/wiki/Windows_NT_3.1) en 1993) establece que **no hay seguridad en Windows si se otorga a las personas derechos de administrador local**. Los derechos de administrador local brindan la capacidad de omitir todas las configuraciones de MDM y las políticas de grupo de la compañía, tomar la identidad de cualquier usuario que haya iniciado sesión, leer o eliminar cualquier archivo del ordenador (incluso los que tienen un ACL `Deny` explícito) y probablemente lo peor: la capacidad de atacar al resto de los sistemas de la empresa.

El modelo del menor privilegio posible ha ido evolucionando con el tiempo:

![Fuente: Sami Laiho][3]

Los informes de Microsoft del año 2019 confirman lo que se había visto desde el año 2015: un gran porcentaje de las vulnerabilidades críticas en sus productos (Windows, Office, Internet Explorer, Edge) se pueden mitigar si no se tienen permisos de administrador. [BeyondTrust](https://www.beyondtrust.com/) lo resume en este gráfico:

![Fuente: BeyondTrust][4]

### Demo: los privilegios ganan a los permisos

<p></p>

* Un Administrador de dominio inicia sesión en `DC1` (Windows Server 2012 R2)
* Accede a `\\STUDENTPC\C$` y crea un directorio `C:\SECRET` en el ordenador del estudiante
* Cambia los ACL para que únicamente él (y nadie más) pueda acceder
* Además se hace propietario de la carpeta
* El estudiante (que es administrador local de su máquina) inicia sesión en `STUDENTPC' (Windows 10)
* Al intentar acceder se le deniega el permiso :-(
* Pero puede hacerse propietario de la carpeta y cambiar los permisos para poder acceder y crear un fichero dentro :-)
* El Administrador de dominio vuelve a hacerse con la propiedad de la carpeta
* Y crea un ACL para denegar explicíto para denegar el permiso de hacerse propietario de la carpeta al estudiante
* Pero el estudiante sigue pudiendo hacerse con la propiedad porque el grupo de Administradores tiene ese privilegio concedido (`SeTakeOwnershipPrivilege`) :-)

{: .box-note}
**Moraleja**: Los privilegios (los permisos respecto del Sistema Operativo) ganan a los permisos (lo que se puede hacer con objetos: ficheros, registro, etc.)

Mediante el comando `whoami /all` se pueden ver los privilegios de un usuario:

```
Privilege Name                Description                          State
============================= ==================================== ========
SeShutdownPrivilege           Shut down the system                 Disabled
SeChangeNotifyPrivilege       Bypass traverse checking             Enabled
SeUndockPrivilege             Remove computer from docking station Disabled
SeIncreaseWorkingSetPrivilege Increase a process working set       Disabled
SeTimeZonePrivilege           Change the time zone                 Disabled
SeCreateSymbolicLinkPrivilege Create symbolic links                Disabled
```

Y los privilegios de un administrador:

```
Privilege Name                            Description                                                        State
========================================= ================================================================== ========
SeLockMemoryPrivilege                     Lock pages in memory                                               Disabled
SeIncreaseQuotaPrivilege                  Adjust memory quotas for a process                                 Disabled
SeSecurityPrivilege                       Manage auditing and security log                                   Disabled
SeTakeOwnershipPrivilege                  Take ownership of files or other objects                           Disabled
SeLoadDriverPrivilege                     Load and unload device drivers                                     Disabled
SeSystemProfilePrivilege                  Profile system performance                                         Disabled
SeSystemtimePrivilege                     Change the system time                                             Disabled
SeProfileSingleProcessPrivilege           Profile single process                                             Disabled
SeIncreaseBasePriorityPrivilege           Increase scheduling priority                                       Disabled
SeCreatePagefilePrivilege                 Create a pagefile                                                  Disabled
SeBackupPrivilege                         Back up files and directories                                      Disabled
SeRestorePrivilege                        Restore files and directories                                      Disabled
SeShutdownPrivilege                       Shut down the system                                               Disabled
SeDebugPrivilege                          Debug programs                                                     Disabled
SeSystemEnvironmentPrivilege              Modify firmware environment values                                 Disabled
SeChangeNotifyPrivilege                   Bypass traverse checking                                           Enabled
SeRemoteShutdownPrivilege                 Force shutdown from a remote system                                Disabled
SeUndockPrivilege                         Remove computer from docking station                               Disabled
SeManageVolumePrivilege                   Perform volume maintenance tasks                                   Disabled
SeImpersonatePrivilege                    Impersonate a client after authentication                          Enabled
SeCreateGlobalPrivilege                   Create global objects                                              Enabled
SeIncreaseWorkingSetPrivilege             Increase a process working set                                     Disabled
SeTimeZonePrivilege                       Change the time zone                                               Disabled
SeCreateSymbolicLinkPrivilege             Create symbolic links                                              Disabled
SeDelegateSessionUserImpersonatePrivilege Obtain an impersonation token for another user in the same session Disabled
```

Los permisos se asignan mediante `secpol.msc` (**Local Security Policy**) desde `Security Settings > Local Policies > User Rights Assigment`.

![Local Security Policy][5]

### Demo: Sobreescritura de ficheros

<p></p>

* El estudiante que es Administrador local abre un `cmd.exe` elevando los permisos
* Intenta copiar un fichero malicioso: `copy /y c:\temp\write.exe c:\windows\system32`
* Se produce un ACCESS DENIED porque hay protecciones adicionales para los binarios de sistema
* Pero si hace `robocopy.exe c:\temp c:\windows\system32 write.exe /B` el fichero se sobreescribe :-(

{: .box-note}
**Nota**: Esto es debido a que se ha usado `robocopy.exe` en modo **Backup** y el usuario tiene el privilegio `SeRestorePrivilege` que le permite restaurar ficheros y directorios.

### Demo: Impersonar una conexión por RDP

<p></p>

* El estudiante llama al _Helpdesk_ diciendo que tiene un problema con su equipo
* El estudiante bloquea la sesión
* _Heldesk_ se conecta a su equipo mediante RDP (`mstsc.exe`) usando la cuenta del Administrador de dominio
* Cuando acaba no cierra la sesión sino únicamente cierra la ventana del cliente RDP
* Los administradores locales tienen el privilegio `SeImpersonatePrivilege` (Impersonate a client after authentication)
* El estudiante ejecuta [Process Hacker 2](https://processhacker.sourceforge.io/)
* Localiza el proceso `explorer.exe` correspondiente a la sesión del Administrador local
* Y ejecuta un comando como ese usuario, por ejemplo `cmd.exe`
* A partir de aquí puede crear persistencia creando un usuario de dominio:

```
net.exe user hacker P@ssw0rd! /add /domain
net.exe group "Domain Admins" hacker /add /domain
```

Si Process Hacker estuviera bloqueado, se podría hacer lo siguiente:

* Ejecutar un CMD como SYSTEM: `psexec.exe -sid cmd.exe`
* Desde el CMD anterior ejecutar el Administrador de Tareas: `taskmgr.exe`
* Acceder a la pestaña de Usuarios
* Conectarse a la sesión del Administrador local (sin conocer el password)

### Demo: Cambiar una GPO

<p></p>

* El Administrador de dominio aplica una política de grupo para permitir RDP
* El estudiante, aún siendo administrador local, no puede cambiar ese valor desde la interfaz GUI
* El estudiante busca la clave del registro modificada por la GPO usando _Group Policy Settings Reference Spreadsheet_:
  * [Windows 10 October 2020 Update (20H2)](https://www.microsoft.com/en-us/download/102158) [*.admx](https://www.microsoft.com/en-us/download/102157)
  * [Windows 10 May 2020 Update (2004)](https://www.microsoft.com/en-us/download/101451) [*.admx](https://www.microsoft.com/en-us/download/101445)
  * [Microsoft Security Compliance Toolkit 1.0](https://www.microsoft.com/en-us/download/55319)
* También podría haber usado la web [Group Policy Search](https://gpsearch.azurewebsites.net/)
* Busca la información sobre _Allow users to connect remotely by using Remote Desktop Services_  
* Obtiene la clave `HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services!fDenyTSConnections`
* Ejecuta `regedit.exe`, la localiza y borra el valor
* El estudiante ya puede modificar la configuración desde la interfaz gráfica

{: .box-note}
**Nota**: Alguien podría decir que cuando se volviera a aplicar la política se aplicaría de nuevo el cambio, pero como la política no se ha modificado no lo haría (tienen la misma versión). Este comportamiento se puede cambiar habilitando la política **Configure registry policy processing** con la opción **Process event if the Group Policy objects have not changed**.

### Demo: Hacerse Administrador de dominio

<p></p>

* El estudiante crea una tarea programada que se ejecutará como `DOMAIN\Administrator` y la opción **Run with highest privileges** (no hace falta especificar la contraseña) 
* El disparador de la tarea será **At log on**
* El comando de la tarea será `net.exe` con los parámetros `group "Domain Admins" student_a /add /domain`
* _Heldesk_ se conecta a su equipo mediante RDP (`mstsc.exe`) usando la cuenta del Administrador de dominio

{: .box-warning}
**Atención**: UAC no es un sistema de seguridad según Microsoft. Por ejemplo, sirve para preguntar por la elevación de los privilegios. De hecho el valor normal de UAC es **Prompt for consent for non-Windows binaries**. Sami usa una _baseline_ y tiene este valor en **Prompt for credentials on the secure desktop**.

![UAC Default][6]

Para ejecutar una aplicación (por ejemplo `cmd.exe`) con todos los privilegios se puede hacer clic mientras se pulsa `CTRL+Shift` (equivale al 'Ejecutar como Adminstrador'). Pero con el valor por defecto, ésto está relajado y se pueden tener los máximos privilegios haciendo lo siguiente:

* Buscar `taskschd.msc` con la lupa y ejecutarlo
* Al ser un binario de Microsoft se habrá elevado automáticamente sin preguntar con UAC
* Se escoge cualquier tarea al azar y se accede a la pestaña del programa que ejecuta
* Se usa el botón para navegar hasta `c:\windows\system32`
* Se localiza `cmd.exe`
* Botón derecho encima del ejecutable y seleccionar Abrir
* El Símbolo del sistema que se ha abierto tiene todos los privilegios (se puede comprobar mediante `whoami /all`)

{: .box-note}
**Nota**: Aunque Sami no recomienda iniciar sesión como Administrador, si realmente hay que hacerlo, aconseja usar el valor de UAC **Prompt for credentials on the secure desktop**. Y si tenemos miedo de los _keyloggers_ se puede usar **Prompt for consent on the secure desktop**. El primer valor sería interesante en un servidor o en una estación PAW porque para elevar es necesario escribir la contraseña.

## Soluciones RunAs

El uso de RunAs es tan sencillo como crear un acceso directo a la aplicación que se quiera utilizar. Por ejemplo, aquí se elevaría `notepad.exe` (en función de los ajustes que se hayan definido en las políticas de UAC comentadas anteriormente) usando las credenciales grabadas del usuario 'student_a':

```
runas.exe /savecred /user domain\student_a c:\windows\system32\notepad.exe
```

El problema es que se puede cambiar ese acceso directo por cualquier otro ejecutable y se podrá ejecutar sin que pida la contraseña (por ejemplo a un simple `cmd.exe`). Otro problema es que las aplicaciones ejecutadas de esta manera se están ejecutando en otro perfil de usuario y, por tanto, no tienen acceso a los mismos [_known folders_](https://docs.microsoft.com/en-us/onedrive/redirect-known-folders) (Documents, Pictures, Desktop).

Para "solucionar" este problema se comenzó a usar [**eRunAs**](https://sourceforge.net/projects/erunas/) para encriptar la línea de comandos y que fuera más complicado cambiarla. Ahora bien, Sami indica que esta solución y otras como [**Sudo for Windows (sudowin)**](https://sourceforge.net/projects/sudowin/), [**RunAsSpc**](https://robotronic.de/runasspcen.html), etc. no sirven ya que todas usan la función [`CreateProcessWithLogonW`](https://docs.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-createprocesswithlogonw).

Además, en su conferencia "[Zero Admins - Zero Problems](https://channel9.msdn.com/Events/Ignite/2015/BRK2335)" (Microsoft Ignite 2015, BRK2335), Sami explicó cómo utilizar [**API Monitor**](http://www.rohitab.com/apimonitor) para obtener las llamadas a `CreateProcessWithLogonW` y obtener el usuario, el dominio y la contraseña usadas. [`MakeMeAdmin`](https://github.com/pseymour/MakeMeAdmin) funciona de forma diferente.

Usando `taskmgr.exe` se puede conocer si un proceso se está ejecutando de forma elevada añadiendo la columna `Elevated` en la pestaña `Details`:

![Task Manager (Elevated)][7]

Mediante el uso de herramientas como **Process Monitor** (`procmon.exe`) se pueden buscar los errores `ACCESS DENIED` para intentar relajar los permisos sobre esos ficheros, claves de regitro, etc. que permitan no tener que ejecutar las aplicaciones con permisos de Administrador.

Otra herramienta interesante es [**Application Compatibility Toolkit**](https://docs.microsoft.com/en-us/windows/win32/win7appqual/application-compatibility-toolkit--act-) (ACT) que incluye [**Standard User Analyzer**](https://docs.microsoft.com/en-us/windows/win32/win7appqual/standard-user-analyzer--sua--tool-and-standard-user-analyzer-wizard--sua-wizard-) (SUA) para analizar las llamadas que puedan causar incompatiblidades con el UAC.

Finalmente, [Aaron Margosis](https://twitter.com/AaronMargosis) creó una herramienta llamada [**LUA Buglight**](https://techcommunity.microsoft.com/t5/windows-blog-archive/lua-buglight-2-3-with-support-for-windows-8-1-and-windows-10/ba-p/701459) que permite identificar problemas con los permisos de Administrador en las aplicaciones.

{: .box-note}
**Nota**: En esta primera parte se ha visto cómo abusar de los derechos de administrador y se ha explicado que la eliminación de estos extiende la vida útil de la instalación del sistema operativo y reduce los tickets.

## Solución Real

{: .box-note}
**Solución**: La solución real es dejar de dar permisos a los usuarios o a los ordenadores y **dar permisos a los procesos y tareas**. La seguridad tiene un _trade-off_ respecto a la usabilidad y el coste tal como se muestra en la siguiente figura.

[![Fuente: eimaung.com][8]](https://eimaung.com/info-sec)

Hay diferentes soluciones en el mercado para hacer ésto: [**BeyondTrust**](https://www.beyondtrust.com/), [**PolicyPak**](https://www.policypak.com/), [**Centero Carillon**](https://centero.fi/en/centero-carillon-for-access-right-management/), etc. **Si no se puede pagar ninguna de ellas habrá que hacerlo todo de forma manual :-(**

La historia detrás de BeyondTrust es muy curiosa. Había 3 compañias **Avecto** (Defendpoint), **BeyondTrust** (PowerBroker) y **Bomgar**. Bomgar compró Avecto y después BeyondTrust. El producto que se está comprando es realmente Defendpoint con el nombre de BeyondTrust. Ahora mismo es la mejor solución pero también la más cara ;-)

PolicyPak, una compañía creada por [Jeremy Moskowitz](https://twitter.com/jeremymoskowitz), acaba de comenzar en este mercado utilizando su conocimiento en la gestión de políticas del Active Directory. Aún está muy por detrás de BeyondTrust aunque es mucho más barato. Por ejemplo, con BeyondTrust se puede permitir que un usuario pueda ejecutar `ncpa.cpl` y cambiar la configuración de red ya que puede capturar la comunicación de objetos COM y elevarla. En cambio en PolicyPak hay que usar una aplicación a parte y elevarla.

BeyondTrust puede elevar las aplicaciones cambiando el token de los usuarios, es decir, que no habrá problemas de doble identidad que se han comentado anteriormente. Esto se consigue mediante **BeyondTrust Privilege Management Hook** (`C:\Program Files\Avecto\Privilege Guard Client\PGHook.dll`). Como se puede ver a continuación, `mmc.exe` está elevado, tiene el token `BUILTIN\Administrators` y, en cambio, `outlook.exe` sigue ejecutándose como usuario:

![Fuente: Sami Laiho][9]]

Con BeyondTrust se puede afinar mucho qué se permite elevar y qué no. Por ejemplo, se puede indicar que deja gestionar Hyper-V (`virtmgmt.msc`) pero no cambiar los dispositivos del ordenador (`devmgmt.msc`). También se puede llegar a indicar qué operaciones se permiten dentro del panel de control de red, etc. La siguiente pantalla muestra las opciones disponibles a día de hoy:

![BeyondTrust Group Policies][10]

Permite usar _allowlisting_ con las aplicaciones de la Microsoft Store por lo que, en este sentido, es mejor que [**AppLocker**](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/applocker/what-is-applocker). Además ya tiene plantillas predefinidas para detectar todo tipo de aplicaciones y/o operaciones que sea necesario elevar:

![BeyondTrust Templates][11] ![BeyondTrust Templates][12]

A la hora de agregar una aplicación, ésta se puede identificar por nombre, directorio, _hash_ y muchísimas más opciones que permiten afinar que realmente se deja elevar una aplicación determinada y no otra similar:

![BeyondTrust Applications][13]

Una opción muy interesante es "**Force standard user on File Open/Save dialogs**" para evitar que se pueden ejecutar comandos elevados desde allí tal como se mostró en un ejemplo anterior. No todas las aplicaciones en este mercado pueden hacerlo. También es interesante la opción para permitir/denegar que todos los procesos hijos de la aplicación elevada también se eleven. Por ejemplo, si se eleva `cmd.exe` sin esta opción, el comando `net stop spooler` no funcionará. En cambio, con la opción activada, se podrá parar el servicio de impresión ;-)

Otra opción interesante es permitir la elevación de una aplicación únicamente si un supervisor introduce la contraseña. Es una elevación _over the shoulder_ y lo utilizan en su fábrica para permitir el uso de **WireShark** a los técnicos que revisan las máquinas. También se podría permitir la elevación de la aplicación pidiendo un código que alguien te proporciona (en el siguiente ejemplo se utiliza en la aplicación `notepad.exe`). Ese código puede servir para una vez, hasta que finalice la sesión o ser válido para siempre:

![Fuente: Sami Laiho][14] ![Fuente: Sami Laiho][15]

Cuando se empieza con BeyondTrust, es interesante instalarlo en modo **Monitor**. Así va mirando qué operaciones requieren elevación y lo escribe en el registro de eventos. Después se puede usar WEF para centralizarlos. De esta manera se puede ver qué habría que configurar más tarde en producción cuando se quiten los permisos de administrador a los usuarios. De hecho, la consola de BT permite crear una regla leyendo estos eventos directamente, lo cual ahorra muchísimo trabajo.

La creación de tokens personalizados permitiría hacer cosas como **poder acceder a la base de datos únicamente con esta aplicación** (cosa que con permisos de Administrador podrías hacer con cualquier aplicación). Estos tokens contienen los grupos a los que pertenecerá el usuario, los privilegios que tendrá, el nivel de integridad de UAC y los derechos de acceso a los procesos. Es muy potente.

La mayoría de empresas que deciden no usar BeyondTrust lo hacen por un motivo económico (25-30€ por usuario) y no técnico (ya que resulve perfectamente todos los problemas relacionados con los permisos de administrador).

PolicyPak es mucho más económico pero hay que tener en cuenta que no és únicamente una solución para controlar estos permisos (**Least Privilege Manager**) sino un pack de productos.

![PolicyPak Suite][16]

PolicyPak cada vez se está acercando más a lo que puede hacer BeyondTrust y tiene muchas funcionalidades para permitir un funcionamento con los menores privilegios posibles:

![PolicyPak Least Privilege Manager][17] 

Aunque no puede capturar las llamadas COM como hace BeyondTrust, tiene unas aplicaciones especiales para permitir el uso privilegiado de los _applets_ de red, de impresoras y de programas instalados:

![PolicyPak Network/Printers/Programs][18]

Otras aplicaciones similares:

* [Ivanti Application Control](https://www.ivanti.com/products/application-control), antiguamente AppSense
* [CyberArk Endpoint Privilege Manager](https://www.cyberark.com/products/privileged-account-security-solution/endpoint-privilege-manager/), antiguamente Viewfinity

## 20 Security Controls

El [Center for Internet Securiy](https://www.cisecurity.org/) (CIS) detallan los [20 controles de seguridad](https://www.cisecurity.org/controls/cis-controls-list/) [[PDF]((https://learn.cisecurity.org/cis-controls-download))] que se pueden implementar para bloquear el 99,99% de las amenazas de seguridad:

| &nbsp; | **Controles básicos de CIS** |
| [1](https://www.cisecurity.org/controls/inventory-and-control-of-hardware-assets) | Inventario y control de activos de hardware |
| [2](https://www.cisecurity.org/controls/inventory-and-control-of-software-assets) | Inventario y control de activos de software |
| [3](https://www.cisecurity.org/controls/continuous-vulnerability-management/) | Gestión continua de la vulnerabilidad |
| [4](https://www.cisecurity.org/controls/controlled-use-of-administrative-privileges) | Uso controlado de privilegios administrativos |
| [5](https://www.cisecurity.org/controls/secure-configuration-for-hardware-and-software-on-mobile-devices-laptops-workstations-and-servers) | Configuración segura de hardware y software en dispositivos móviles, computadoras portátiles, estaciones de trabajo y servidores |
| [6](https://www.cisecurity.org/controls/maintenance-monitoring-and-analysis-of-audit-logs) | Mantenimiento, seguimiento y análisis de registros de auditoría |
| &nbsp; | **Controles CIS fundamentales** |
| [7](https://www.cisecurity.org/controls/email-and-web-browser-protections) | Protecciones de correo electrónico y navegador web |
| [8](https://www.cisecurity.org/controls/malware-defenses) | Defensas contra malware |
| [9](https://www.cisecurity.org/controls/limitation-and-control-of-network-ports-protocols-and-services) | Limitación y control de puertos, protocolos y servicios de red |
| [10](https://www.cisecurity.org/controls/data-recovery-capability) |  Capacidades de recuperación de datos |
| [11](https://www.cisecurity.org/controls/secure-configuration-for-network-devices-such-as-firewalls-routers-and-switches) |  Configuración segura para dispositivos de red, como firewalls, enrutadores y conmutadores |
| [12](https://www.cisecurity.org/controls/boundary-defense) |  Defensa de fronteras (_Boundary defense_) |
| [13](https://www.cisecurity.org/controls/data-protection) |  Protección de datos |
| [14](https://www.cisecurity.org/controls/controlled-access-based-on-the-need-to-know) |  Acceso controlado basado en la necesidad de saber (_Need to know_) |
| [15](https://www.cisecurity.org/controls/wireless-access-control) |  Control de acceso inalámbrico |
| [16](https://www.cisecurity.org/controls/account-monitoring-and-control) |  Seguimiento y control de cuentas |
| &nbsp; | **Controles CIS organizacionales** |
| [17](https://www.cisecurity.org/controls/implement-a-security-awareness-and-training-program) |  Implementar un programa de capacitación y concienciación sobre seguridad |
| [18](https://www.cisecurity.org/controls/application-software-security) |  Seguridad del software de aplicación |
| [19](https://www.cisecurity.org/controls/incident-response-and-management) |  Respuesta y gestión de incidentes |
| [20](https://www.cisecurity.org/controls/penetration-tests-and-red-team-exercises) |  Pruebas de penetración y ejercicios del equipo rojo |

## Contacto

Se puede contactar con Sami Laiho en Twitter y en su blog de forma gratuita. También se puede contratar el acceso a sus _training_ en su _dojo_ particular:

![Sami Laiho Contact][19]

<p></p>

[1]: /assets/img/blog/2020-12-12_image_1.png "Fuente: Microsoft"
[2]: /assets/img/blog/2020-12-12_image_2.png "Grupo Administradores"
[3]: /assets/img/blog/2020-12-12_image_3.png "Fuente: Sami Laiho"
[4]: /assets/img/blog/2020-12-12_image_4.png "Fuente: BeyondTrust"
[5]: /assets/img/blog/2020-12-12_image_5.png "Local Security Policy"
[6]: /assets/img/blog/2020-12-12_image_6.png "UAC Default"
[7]: /assets/img/blog/2020-12-12_image_7.png "Task Manager (Elevated)"
[8]: /assets/img/blog/2020-12-12_image_8.png "Fuente: eimaung.com"
[9]: /assets/img/blog/2020-12-12_image_9.png "Fuente: Sami Laiho"
[10]: /assets/img/blog/2020-12-12_image_10.png "BeyondTrust Group Policies"
[11]: /assets/img/blog/2020-12-12_image_11.png "BeyondTrust Templates"
[12]: /assets/img/blog/2020-12-12_image_12.png "BeyondTrust Templates"
[13]: /assets/img/blog/2020-12-12_image_13.png "BeyondTrust Applications"
[14]: /assets/img/blog/2020-12-12_image_14.png "Fuente :Sami Laiho"
[15]: /assets/img/blog/2020-12-12_image_15.png "Fuente :Sami Laiho"
[16]: /assets/img/blog/2020-12-12_image_16.png "PolicyPak Suite"
[17]: /assets/img/blog/2020-12-12_image_17.png "PolicyPak Least Privilege Manager"
[18]: /assets/img/blog/2020-12-12_image_18.png "PolicyPak Network/Printers/Programs"
[19]: /assets/img/blog/2020-12-12_image_19.png "Sami Laiho Contact"
