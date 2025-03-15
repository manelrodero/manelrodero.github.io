---
layout : post
blog-width: true
title: 'Uso de GrapheneOS con Google Play Services'
date: '2023-12-11 14:33:55'
published: true
tags:
- Android
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2023-12-11_cover.png"
thumbnail-img: ""
---

La semana pasada [instalé GrapheneOS en un Google Pixel 7a](instalacion-de-grapheneos-en-un-google-pixel-7a) y durante los últimos días he estado "jugando" con el teléfono e instalando algunas aplicaciones para ver su comportamiento.

Aunque la idea de todos aquellos que instalan este sistema operativo es librarse de Google usando F-Droid o Aurora Store para instalar las aplicaciones, yo he estado probando [**Sandboxed Google Play**](https://grapheneos.org/usage#sandboxed-google-play){:target="_blank"}. En este modo, Google Play se instala como una aplicación más y no recibe permisos privilegiados ni interactúa con el sistema operativo como hace en un Android tradicional.

También he probado los [perfiles de usuario](https://grapheneos.org/features#improved-user-profiles){:target="_blank"} para intentar separar las aplicaciones por tipo (las bancarias, las de redes sociales, las que necesitan los servicios de Google Play, etc.). Pero no me ha acabado de convencer su uso porque requiere configurar los diferentes aspectos del sistema en cada uno de ellos y porque, aunque existe la posibilidad de [reenviar las notificaciones](https://grapheneos.org/features#notification-forwarding){:target="_blank"} de un perfil a otro, en muchas ocasiones no me ha funcionado.

Al final he decidido usar **un único perfil de usuario** tal como he venido haciendo en todos los teléfonos anteriores sin ningún problema aparente más allá de los aspectos relacionados con la privacidad y seguridad de las aplicaciones.

# Sandboxed Google Play

Google Play está formado por tres aplicaciones:

* Google Services Framework (GSF)
* Google Play services
* Google Play Store

Para instalarlas hay que acceder a la aplicación `Apps` (donde veremos que únicamente hay instaladas 5 aplicaciones, la propia `Apps`, `Auditor`, `Camera`, `PDF Viewer` y `Vanadium`), seleccionar la aplicación **`Google Play services`** y pulsar sobre el botón de instalación.

Al instalar `Google Play services`, la tienda de aplicaciones de GrapheneOS se encarga de instalar todas las dependencias. Para cada aplicación, será necesario confirmar su instalación y desactivar el permiso de red si no es necesario para su funcionamiento.

* Google Play Store (Install + Network)
* GmsCompatConfig (Update)
* Servicios de Google Play (Install + Network)
* Framework de servicios de Google (Install + Network)

Al ejecutar por primera vez Google Play aparece una notificación sobre un problemas que hay que resolver para que todo funcione correctamente:

```plaintext
GmsCompat:
Missing optional permission
Push notifications will be delayed, because Play services app is not allowd to alwayas run in the background.
Tap to resolve.
```

Al pulsar en la notificación aparece la pregunta "¿Permitir que la aplicación se ejecute siempre en segundo plano?" a la cual se responderá **Permitir**.

{: box-note}
**Nota**: Al permitir que los Servicios de Google Play siempre se estén ejecutando en segundo plano, la duración de la batería podría reducirse. Se puede cambiar esta opción desde `Ajustes > Aplicaciones`.

Otros problemas que hay que resolver pulsando en la notificación correspondiente son:

* Servicios de Google Play intenta acceder a los sensores &rarr; **Denegar**
* Google Play Store intenta mostrar notificaciones &rarr; **Permitir**
* Google Play Store neceita actualizar Google Play Store &rarr; **Permitir**

Al intentar actualizar Google Play Store, aparece el mensaje "_Por tu seguridad, de momento tu teléfono no puede instalar aplicaciones desconocidas de esta fuente_". Se puede autorizar la instalación desde esta fuente desde `Ajustes > Instalar aplicaciones desconocidas > Google Play`.

A continuación se puede iniciar sesión en Google Play para tener acceso a las aplicaciones que se hayan comprado, las aplicaciones familiares, etc. Después de introducir la dirección de correo electrónico y la contraseña, se pide confirmación del acceso en una notificación _push_ enviada al teléfono antiguo.

Finalmente, se soluciona un último problema relacionado con los **contactos** en los `Servicios de Google Play`:

```plaintext
GmsCompat:
Missing optional permission
To sync contacts from a Google account, Play services need the Contacts permission.
Tap to open settings.
```

# Gmail (com.google.android.gm)

Dado que aún utilizo [**Gmail**](https://play.google.com/store/apps/details?id=com.google.android.gm){:target="_blank"}, ésta será la primera aplicación que se instalará para comprobar el correcto funcionamiento de la tienda y las notificaciones.

La instalación se realiza de la forma habitual. Se accede a Google Play Store, se busca Gmail y se pulsa el botón correspondiente. La interfaz de usuario de GrapheneOS confirmará la instalación y los permisos de red (que, obviamente, necesita para poder acceder al correo que está en la nube de Google).

Al ejecutar por primera vez la aplicación, habrá que proporcionar/denegar permisos:

* Notificaciones &rarr; **Permitir**
* Sensores &rarr; **No permitir**

# Microsoft Outlook (com.microsoft.office.outlook)

Ahora se necesita una segunda aplicación de correo para probar el envío y recepción de notificaciones. Para ello instalo [**Microsoft Outlook**](https://play.google.com/store/apps/details?id=com.microsoft.office.outlook){:target="_blank"} y la configuro usando mi cuenta Microsoft.

Los permisos otorgados son los siguientes:

* Network &rarr; **Permitir**
* Notificaciones &rarr; **Permitir**

A continuación se envían dos mensajes (uno de Gmail a Outlook y otro de Outlook a Gmail) y se observa la pantalla del teléfono para comprobar si llegan las notificaciones.

Se observa que éstas llegan correctamente **aunque un poco más tarde** que en mi teléfono antiguo, el Xiami Mi Mix 2S. En la parte superior izquierda de la pantalla aparecen los iconos de Outlook y Gmail. En los iconos de las aplicaciones aparece un punto de color y también se escucha el sonido de cada notificación.

{: .box-note}
**Nota**: La notificación de Outlook utiliza un "Sonido proporcionado por la aplicación". La notificación de Gmail utiliza el "Sonido de notificación predeterminado".

# Google Maps (com.google.android.apps.maps)

A continuación se instala la aplicación [**Google Maps**](https://play.google.com/store/apps/details?id=com.google.android.apps.maps){:target="_blank"} mediante la cual se podrá comprobar el funcionamiento de la [**Ubicación**](https://grapheneos.org/usage#sandboxed-google-play-configuration){:target="_blank"}.

Por defecto, GrapheneOS redirige las peticiones de _location_ a una reimplementación propia de los servicios de geolocalización. Si se quieren utilizar los servicios de Google Play (por ejemplo, para tener una localización aproximada cuando hay cobertura de GPS) habría que desactivar la opción `Reroute location requests to OS APIs` y alguna configuración más.

{: .box-note}
**Nota**: Se recomienda leer detenidamente la documentación si se necesita cambiar esta configuración en GraheneOS.

Los permisos otorgados son los siguientes:

* Network &rarr; **Permitir**
* Sensores &rarr; **Permitir**
* Ubicación &rarr; **Permitir** (Mientras se usa la aplicación)

# Google Calendar (com.google.android.calendar)

La aplicación [**Google Calendar**](https://play.google.com/store/apps/details?id=com.google.android.calendar){:target="_blank"} se instala con los siguientes permisos:

* Network &rarr; **Permitir**
* Calendario &rarr; **Permitir**
* Contactos &rarr; **Permitir**
* Notificaciones &rarr; **Permitir**

Una vez instalada, en `Ajustes > Contraseñas y cuentas > Cuentas de propietario > Sincronización de cuenta` se puede comprobar que se han añadido correctamente los elementos a sincronizar:

* Calendario
* Contactos
* Gmail
* Google Calendar
* Tareas en Calendar

Al ejecutarla por primera vez, recibimos las notificaciones visuales y de sonido habituales para los próximos eventos del calendario.

# Google Drive (com.google.android.apps.docs)

La aplicación [**Google Drive**](https://play.google.com/store/apps/details?id=com.google.android.apps.docs){:target="_blank"} se instala con los siguientes permisos:

* Network &rarr; **Permitir**

# Referencias

* [Exodus. The privacy audit platform for Android applications](https://reports.exodus-privacy.eu.org/en/){:target="_blank"}
