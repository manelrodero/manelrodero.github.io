---
layout : post
blog-width: true
title: 'Instalación de GrapheneOS en un Google Pixel 7a'
date: '2023-12-08 09:11:28'
published: true
tags:
- Android
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2023-12-08_cover.png"
thumbnail-img: ""
---

Hace unos días [compré un Google Pixel 7a](bienvenido-google-pixel-7a) para sustituir a mi "viejo" Xiaomi Mi Mix 2S del año 2019, que ya comenzaba a trabarse con algunas aplicaciones.

Aunque la idea inicial era usarlo unos meses con la versión de Android proporcionada por Google, como me conozco y sé que me dará pereza reinstalar el teléfono cuando lo tenga todo bien configurado, voy a instalar directamente **GrapheneOS**.

# GrapheneOS

[**GrapheneOS**](https://grapheneos.org/){:target=_blank} es un sistema operativo móvil de código abierto basado en Android, enfocado en la privacidad y seguridad para dispositivos **Google Pixel** seleccionados.

GrapheneOS se centra en la investigación y desarrollo de tecnología de privacidad y seguridad, incluyendo mejoras sustanciales en el aislamiento de aplicaciones, mitigaciones de explotación y el modelo de permisos.

Algunas de sus características más destacadas son las siguientes:

| **Característica** | **Descripción** |
|----------------|-------------|
| Compatibilidad con apps de Android | Permite el uso de aplicaciones de Android manteniendo un enfoque en la privacidad y seguridad |
| Sandboxed Google Play | Posibilidad de instalar servicios de Google en una versión aislada y sandboxed |
| Navegador Vanadium | Un navegador web y WebView basado en Chromium endurecido específicamente para GrapheneOS |
| Auditor app y servicio de atestiguación | Una aplicación basada en hardware para la verificación local y remota de dispositivos |
| Cámara GrapheneOS | Una aplicación de cámara moderna y enfocada en la privacidad/seguridad |
| Visor de PDF GrapheneOS | Un visor de PDF minimalista y enfocado en la seguridad |
| Backups encriptados | Soporte para copias de seguridad encriptadas |
| Indicador de acceso a datos de ubicación | Indicador visual cuando las aplicaciones acceden a los datos de ubicación |
| Permisos revocables de red y sensores | Interruptores de permisos para el acceso a la red y sensores para cada aplicación instalada |
| Privacidad por defecto | Configuraciones predeterminadas que priorizan la privacidad del usuario |

# Google Pixel

Aunque parezca una contradicción utilizar un teléfono de Google para "aumentar la privacidad", ésto es debido a que los teléfonos Google Pixel tienen una serie de características de seguridad que no están disponibles en otras teléfonos:

| **Característica** | **Descripción** |
|----------------|-------------|
| Seguridad de hardware sólida | Incluye el chip de seguridad Titan M2 y el núcleo de seguridad Tensor para proteger contra ataques físicos y digitales |
| Soporte de sistema operativo alternativo | Permite el uso de claves de firma de usuario y cuenta con Android Verified Boot 2.0 para garantizar la integridad del sistema operativo |
| Soporte a largo plazo | Ofrece actualizaciones de seguridad regulares para mantener el dispositivo protegido a lo largo del tiempo. |
| Funciones de seguridad de vanguardia | Incorpora tecnologías como el etiquetado de memoria para proteger contra vulnerabilidades y ataques |
| Superficie de ataque estrecha | Diseñado con múltiples capas de seguridad para minimizar los riesgos y proteger la privacidad del usuario |
| Autenticación integrada | Funciones de autenticación segura como el desbloqueo facial y el lector de huellas dactilares |
| Inteligencia en el propio dispositivo | Utiliza modelos de aprendizaje automático en el dispositivo para funciones de privacidad y seguridad |

A la hora de comprar un teléfono Google Pixel hay que asegurarse de comprar un dispositivo que no esté bloqueado por el operador para funcionar únicamente en su red (es decir, que no sea **_carrier-locked_**) y que permita habilitar la opción **_OEM unlocking_** para poder instalar otros sistemas operativos (es decir, que no sea **_bootloader-locked_**).

# Instalación de GrapheneOS

La instalación de GrapheneOS se puede realizar usando la [herramienta `fastboot` desde CLI](https://grapheneos.org/install/cli){:target=_blank} o, como haré yo, usando el [**instalador basado en WebUSB**](https://grapheneos.org/install/web){:target=_blank} que es bastante más sencillo.

## Pre-requisitos

Para poder instalar GrapheneOS usando el instalador web es necesario cumplir los siguientes requisitos:

* 2GB de memoria RAM libre y 32GB de espacio de almacenamiento disponible
* Teléfono Google Pixel que pueda habilitar el _OEM unlocking_
* Cable USB-C para conectarlo al ordenador (en mi caso utilizaré un cable USB-C/USB-A)
* Sistema operativo del ordenador y navegador completamente actualizados (en mi caso utilizaré Microsoft Edge en Windows 11)
* Teléfono actualizado a la última versión del firmware

## Actualización firmware

Para actualizar el teléfono se realiza una configuración inicial básica del mismo para conectarlo a la red Wi-Fi y descargar la última actualización del sistema disponible ([Android 14](bienvenido-google-pixel-7a) a día de hoy):

* `Ajustes` &rarr; `Sistema` &rarr; `Actualización del sistema`

Una vez actualizado, es recomendable realizar un restablecimiento al estado de fábrica antes de continuar con la instalación de GrapheneOS:

* `Ajustes` &rarr; `Sistema` &rarr; `Opciones de restablecimiento` &rarr; `Volver al estado de fábrica (borrar todo)`

{: .box-note}
**Nota**: Después de restablecer el teléfono, será necesario volver a realizar una configuración inicial básica para volver a conectarlo a la red Wi-Fi y poder realizar los pasos siguientes.

## Habilitar desbloqueo de OEM

Para poder habilitar el desbloqueo de OEM (_Original Equipment Manufacturer_), que permite desbloquear el _bootloader_ del teléfono y cargar un nuevo sistema operativo, es necesario habilitar las **opciones para desarrolladores** pulsando repetidas veces sobre el número de compilación:

* `Ajustes` &rarr; `Información del teléfono` &rarr; `Número de compilación`

Una vez activadas, se puede acceder a ellas para **habilitar el desbloqueo de OEM**:

* `Ajustes` &rarr; `Sistema` &rarr; `Opciones para desarrolladores` &rarr; `Desbloqueo de OEM` &rarr; **Habilitar**

{: .box-note}
**Nota**: Para habilitar esta opción es necesario estar conectado a Internet y aceptar la advertencia sobre los posibles peligros de desbloquear el _bootloader_.

## Iniciar en el modo _bootloader_

Para iniciar el teléfono en la interfaz de _bootloader_ hay que mantener pulsado el botón `Volume Down` mientras se enciende el teléfono con el botón `Power`.

Si todo funciona correctamente, al cabo de unos segundos aparecerá una pantalla negra con un triángulo rojo indicando **Fastboot Mode** y mostrando información sobre el teléfono:

```
Product revision: lynx MP1.0 B0
Bootloader version: lynx-14.0-10529422
Baseband version: g5300q-230626-230818-b-10679446
Serial number: **************
Secure boot: PRODUCTION
NOS production: yes
DRAM: 8GB Samsung LPDDR5
UFS: 128GB Micron
Device state: locked
Boot slot: b
Enter reason: vol down pressed
UART: disabled
```

{: .box-note}
**Nota**: El estado del dispositivo se mostrará con el texto `locked` en color verde para indicar que es el valor correcto para un dispositivo.

## Conectar el teléfono

En este momento hay que conectar el teléfono al ordenador con el cable USB-C. Si todo va bien, en el "Administrador de Dispositivos" (`devmgmt.msc`) de Windows debería aparecer un **Pixel 7a** en la rama **Other devices**.

A continuación se procede a descargar e instalar el [**Google USB Driver**](https://developer.android.com/studio/run/win-usb){:target=_blank} para este teléfono:

* Botón derecho en `Pixel 7a`
* Seleccionar la opción `Update driver`
* Seleccionar la opción `Browse my computer for drivers`
* Seleccionar la opción `Search for drivers in this location` (escoger el directorio donde esté el fichero `android_winusb.inf`)
* En la pregunta _Would you like to install this device software?_
  * Name: Google, Inc.
  * Publisher: Google LLC
* **Desmarcar** la opción `Always trust software from "Google LCC"?`
* Pulsar el botón `Install`

Si la instalación del _driver_ se realiza correctamente, el teléfono debería cambiar a **Android Bootloader Interface** y aparecer en la rama **Android Device**.

## Desbloquear el _bootloader_

Para desbloquear el _bootloader_ únicamente hay que pulsar el botón azul `Unlock bootloader` de la página web de GrapheneOS.

{: .box-note}
**Nota**: La primera vez que se pulsa uno de estos botones azules, el navegador solicita permiso para conectar con el dispositivo "Pixel 7a" a través de [**WebUSB**](https://developer.mozilla.org/en-US/docs/Web/API/WebUSB_API){:target=_blank}.

A continuación, se usan los botones de volúmen para seleccionar la opción `Unlock the bootloader` y se pulsa el botón `Power` para confirmar esta acción en el teléfono.

Si todo funciona correctamente, el estado del dispositivo pasará a ser `unlocked` y se mostrará el texto en color rojo para indicar que es un valor incorrecto para un dispositivo. ¡Lo sabemos!

## Descargar la imagen de fábrica

Después se pulsa el botón azul `Download release` para descargar la imagen de fábrica de GrapheneOS para mi teléfono. Este proceso puede durar varios minutos dependiendo de la velocidad de la conexión a Internet.

{: .box-note}
**Nota**: A día de hoy, 8 de diciembre de 2023, se descarga el fichero [`lynx-factory-2023120701.zip`](https://releases.grapheneos.org/lynx-factory-2023120701.zip){:target=_blank} correspondiente a la versión [2023120701](https://grapheneos.org/releases#2023120701){:target=_blank} liberada ayer mismo.

## "Flashear" la imagen de fábrica

Al pulsar el botón azul `Flash release` se iniciará la escritura de la imagen de fábrica de GrapheneOS, reemplazando el sistema operativo y borrando todos los datos existentes en el teléfono.

{: .box-note}
**Nota**: Durante este proceso, que dura unos minutos, no debemos interactuar con el teléfono o el ordenador por muy nerviosos que estemos ;-)

El "flasheo" de la imagen de fábrica de GrapheneOS se realizará en diferentes fases:

* Escritura del _firmware_
* Reinicio a la interfaz de _bootloader_
* Escritura del núcleo del Sistema Operativo
* Reinicio en el [modo `fastboot` del espacio de usuario](https://source.android.com/docs/core/architecture/bootloader/fastbootd){:target=_blank}
* Escritura del resto del Sitema Operativo
* Reinicio a la interfaz _bootloader_

## Bloquear el _bootloader_

Una vez finalizado el "flasheo" de la imagen, se procede a bloquear de nuevo el _bootloader_ pulsando el botón azul `Lock bootloader` y confirmando de nuevo la operación en el teléfono.

Si todo funciona correctamente, el estado del dispositivo debería volver a mostrar el texto `locked` en color verde. Esto significará que lo más "difícil" ya está hecho.

# Post-instalación

## Boot

Se puede iniciar GrapheneOS seleccionando la opción `Start` y pulsando el botón `Power` para reiniciar el teléfono.

Al iniciar GrapheneOS por primera vez, aparece el asistente de configuración que hace las siguientes preguntas:

* Pulsar el botón `Start`
* Seleccionar el idioma: **Español (España)**
* Configurar la zona horaria: **Amsterdam (GMT+01:00)**
* Revisar que la fecha y la hora sean correctas
* Saltar la configuración de la `Wi-Fi`
* Saltar la configuración de la tarjeta `SIM`
* Desactivar los servicios de ubicación para las aplicaciones
* Omitir la configuración biométrica de la huella digital
* Omitir la configuración del PIN
* Omitir la restauración de aplicaciones
* Pulsar el botón `Iniciar`

{: .box-note}
**Nota**: Se han omitido algunas configuraciones para ir más rápido durante esta toma de contacto con GrapheneOS pero se podrían completar en este momento sin problema.

## Deshabilitar desbloqueo de OEM

La primera operación importante a realizar después de haber instalado GrapheneOS es volver a deshabilitar el desbloqueo de OEM desde las opciones para desarrolladores (que también se desactivarán ya que no se utilizará el dispositivo para desarrollo de aplicaciones):

* `Ajustes` &rarr; `Información del teléfono` &rarr; `Número de compilación` (pulsar varias veces)
* `Ajustes` &rarr; `Sistema` &rarr; `Opciones para desarrolladores` &rarr; `Desbloqueo de OEM` &rarr; **Deshabilitar*
* Reiniciar el teléfono
* `Ajustes` &rarr; `Sistema` &rarr; `Opciones para desarrolladores` &rarr; **Deshabilitar**
* Reiniciar el teléfono

{: .box-note}
**Nota**: Al cargar un sistema operativo alternativo, el dispositivo muestra un aviso amarillo en el inicio con el ID del sistema operativo alternativo según el [SHA256 de la clave pública de inicio verificada](https://grapheneos.org/install/web#verified-boot-key-hash){:target=_blank}.

# Uso de GrapheneOS

La [**guía de uso de GrapheneOS**](https://grapheneos.org/usage){:target=_blank} es un buen punto de partida para comenzar a utilizar este sistema operativo (por ejemplo, la navegación mediante gestos).

También es interesante leer la página donde se detallan todas las [**características de GrapheneOS**](https://grapheneos.org/features){:target=_blank} para sacarle todo el partido al teléfono.

## Configuración inicial

Aunque los valores por defecto de GrapheneOS son bastante buenos, no está de más realizar algunos cambios para aumentar la seguridad del teléfono antes de instalar aplicaciones.

{: .box-note}
**Nota**: Algunos de estos valores se explican en el vídeo "[Make your Phone more private](https://www.youtube.com/watch?v=wg00QkcpOOM){:target=_blank}" y la guía "[GrapheneOS - Next steps](https://github.com/Scrut1ny/GrapheneOS-Guide){:target=_blank}".

* Redes e Internet
  * SIMs
    * Elegir la SIM y desactivar 2G
  * (Opcional) Modo Avión
  * DNS privado: Automático &rarr; Nombre de host del proveedor de DNS privado: [**Quad9**](https://www.quad9.net/service/service-addresses-and-features){:target=_blank}
* Aplicaciones
  * (Opcional) Aplicaciones predeterminadas
* Notificaciones
  * Notificaciones en pantalla de bloqueo: Oculta conversaciones y notificacioens silenciosas &rarr; No mostrar ninguna notificación
* Pantalla
  * Pantalla de bloqueo
    * Activar pantalla si hay notificaciones: Deshabilitar
  * Tiempo de espera de la pantalla: 15 segundos &rarr; 1 minuto
  * (Opcional) Increase touch sensitivy
* Seguridad
  * Auto reboot: 72 horas &rarr; 12 horas
  * USB peripherals: Allow new USB peripherals when unlocked &rarr; Disallow new USB peripherals
  * Secure app spawning
  * Native code debugging > Block for third-party apps by default: Habilitar
  * Scramble PIN input layout: Habilitar
* Privacidad
  * (Opcional) Gestor de permisos
  * Allow Sensors permission to apps by default: Deshabilitar
* Sistema
  * Fecha y hora
    * Establecer hora automáticamente: Deshabilitar
    * Zona horaria
      * Definir automáticamente: Deshabilitar
      * Región: España
      * Zona horaria: Madrid (GMT+01:00)

## Tiendas de aplicaciones

GrapheneOS, al contrario que otras versiones de Android, se proporciona con un mínimo de aplicaciones instaladas:

* Apps (su "tienda" de aplicaciones)
* Auditor
* Camera
* PDF Viewer
* Vanadium

Desde Apps también se puede instalar [Google Play services](https://play.google.com/store/apps/details?id=com.google.android.gms){:target=_blank}, la capa _software_ que permite a Google [añadir características a Android](https://www.androidauthority.com/google-play-services-1094356/){:target=_blank} sin actualizar el sistema operativo y que es necesaria para utilizar la tienda **Google Play**.

Ahora bien, como no está claro qué tipo de telemetría o _tracking_ realiza Google con esta aplicación, hay quien usa otras tiendas alternativas como [**F-Droid**](https://f-droid.org/){:target=_blank}.

F-Droid es un catálogo instalable de aplicaciones FOSS (software gratuito y de código abierto) para la plataforma Android. El [cliente de F-Droid](https://f-droid.org/F-Droid.apk){:target=_blank} facilita la navegación, la instalación y el seguimiento de las actualizaciones en los dispositivos.

Para instalar algunas aplicaciones que únicamente están en Google Play, se suele instalar [**Aurora Store**](https://f-droid.org/packages/com.aurora.store/){:target=_blank} desde F-Droid. Aurora Store es un _frontend_ de Google Play que respeta la privacidad y permite descargar los APKs de las aplicaciones que se necesiten.

{: .box-note}
**Nota**: Aunque se puede utilizar Aurora Store utilizando una cuenta anónima, es necesario utilizar una uenta de Google para descargar aplicaciones que se hayan pagado.

## Aplicaciones

Algunas de las aplicaciones recomendadas por [Naomi Brockwell](https://www.youtube.com/@NaomiBrockwellTV){:target=_blank} para ser utilizadas con GrapheneOS son las siguientes:

| Aplicación | Tienda | Descripción |
| ---------- | ------ | ----------- |
| [Signal](https://signal.org/es/){:target=_blank} | Oficial | Mensajería privada y segura con cifrado de extremo a extremo |
| [Organic Maps](https://f-droid.org/packages/app.organicmaps/){:target=_blank} | F-Droid | Navegación sin conexión para senderismo y ciclismo con mapas detallados |
| [OSmAnd+](https://f-droid.org/en/packages/net.osmand.plus/){:target=_blank} | F-Droid | Mapas y navegación fuera de línea con énfasis en la privacidad del usuario |
| [DuckDuckGo Privacy Browser](https://f-droid.org/en/packages/com.duckduckgo.mobile.android/){:target=_blank} | F-Droid | Navegador que bloquea rastreadores y proporciona búsquedas privadas |
| [Tor Browser](https://www.torproject.org/){:target=_blank} | Oficial | Navegador enfocado en la privacidad que permite el acceso a la red Tor |
| [Try LBRY](https://f-droid.org/en/packages/ua.gardenapple.trylbry/){:target=_blank} | F-Droid | Plataforma de contenido digital abierta y gratuita con énfasis en la privacidad |
| [NewPipe](https://f-droid.org/en/packages/org.schabi.newpipe/){:target=_blank}  | F-Droid | Cliente de YouTube ligero y sin anuncios con reproducción en segundo plano |
| [AntennaPod](https://f-droid.org/en/packages/de.danoeh.antennapod/){:target=_blank} | F-Droid | Reproductor de podcasts de código abierto y sin anuncios |
| [Mullvad VPN](https://f-droid.org/en/packages/net.mullvad.mullvadvpn/){:target=_blank} | F-Droid | VPN que protege la privacidad y permite navegar de forma segura y privada |
| [ProtonVPN](https://f-droid.org/en/packages/ch.protonvpn.android/){:target=_blank} | F-Droid | VPN segura y privada con una política estricta de no registros |
| [Aegis Authenticator](https://f-droid.org/en/packages/com.beemdevelopment.aegis/){:target=_blank} | F-Droid | Autenticador de dos factores seguro y de código abierto |
| [andOTP](https://play.google.com/store/apps/details?id=org.shadowice.flocke.andotp){:target=_blank} | Google Play | Autenticador de dos factores de código abierto con copias de seguridad encriptadas |
| [Unstoppable Wallet](https://f-droid.org/en/packages/io.horizontalsystems.bankwallet/){:target=_blank} | F-Droid | Cartera de criptomonedas descentralizada y segura |
| [PilferShush Jammer](https://f-droid.org/en/packages/cityfreqs.com.pilfershushjammer/){:target=_blank} | F-Droid | Bloqueador de micrófono para proteger contra la escucha no autorizada |
| [UntrackMe](https://f-droid.org/en/packages/app.fedilab.nitterizeme/){:target=_blank} | F-Droid | Transforma URLs de seguimiento en URLs limpias para proteger la privacidad |
| [Scrambled Exif](https://f-droid.org/en/packages/com.jarsilio.android.scrambledeggsif/){:target=_blank} | F-Droid | Herramienta para eliminar metadatos de imágenes y proteger la privacidad |
| [Shelter](https://f-droid.org/en/packages/net.typeblog.shelter/){:target=_blank} | F-Droid | Crea perfiles de usuario separados para aislar aplicaciones y datos |

# Referencias

* [Vídeo: This Android OS makes sense in 2023](https://www.youtube.com/watch?v=Zy47rVqLx3w){:target=_blank}
* [Vídeo: Make your Phone more private](https://www.youtube.com/watch?v=wg00QkcpOOM){:target=_blank}
* [GrapheneOS - Next steps guide](https://github.com/Scrut1ny/GrapheneOS-Guide){:target=_blank}
* [Vídeo: How to de-Google your phone](https://www.youtube.com/watch?v=rIFBN390clQ){:target=_blank}
* [Vídeo: GrapheneOS: 1 year update and all Apps I use](https://www.youtube.com/watch?v=Ttbbh_IKPgo){:target=_blank}

# Herramientas

* [Android Flash Tool](https://flash.android.com/welcome){:target=_blank}
* [Android SDK Platform Tools](https://developer.android.com/tools/releases/platform-tools){:target=_blank}
