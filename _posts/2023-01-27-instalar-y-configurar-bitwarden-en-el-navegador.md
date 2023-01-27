---
layout : post
blog-width: true
title: 'Instalar y configurar Bitwarden en el navegador'
date: '2023-01-27 14:40:29'
published: true
tags:
- Seguridad
author:
  display_name: Manel Rodero
---

# Introducción

[Bitwarden](https://bitwarden.com/) es un gestor de contraseñas de [código abierto](https://github.com/bitwarden) que se puede utilizar de forma gratuita o mediante una [suscripción personal o familiar](https://bitwarden.com/pricing/) para disponer de algunas características avanzadas (como 2FA o la compartición de cuentas).

Al ser de código abierto, es posible hacer una [instalación local](https://bitwarden.com/help/install-on-premise-linux/) en un servidor Linux o utilizar la implementación alternativa [Vaultwarden](https://github.com/dani-garcia/vaultwarden) mucho más ligera y ejecutable en una simple Raspberry Pi 4.

En cualquiera de los dos casos, es posible acceder al **_vault_ de contraseñas** de varias maneras:

* Accediendo directamente a la _web vault_ a través de la URL [https://vault.bitwarden.com/](https://vault.bitwarden.com/)
* Descargando un cliente de escritorio ([Windows](https://vault.bitwarden.com/download/?app=desktop&platform=windows), [macOS](https://itunes.apple.com/app/bitwarden/id1352778147) y [Linux](https://vault.bitwarden.com/download/?app=desktop&platform=linux))
* Usando una extensión del navegador ([Google Chrome](https://chrome.google.com/webstore/detail/bitwarden-free-password-m/nngceckbapebfimnlniiiahkandclblb), [Mozilla Firefox](https://addons.mozilla.org/firefox/addon/bitwarden-password-manager/), [Microsoft Edge](https://microsoftedge.microsoft.com/addons/detail/jbkfoedolllekgbhcbcoahefnbanhhlh), Safari, etc.) 
* Instalando una aplicación en el móvil para iOS ([AppStore](https://itunes.apple.com/app/bitwarden-free-password-manager/id1137397744?mt=8)) o Android ([Google Play](https://play.google.com/store/apps/details?id=com.x8bit.bitwarden))
* Desde la [línea de comandos](https://bitwarden.com/help/article/cli/)

> **Nota**: si se ha [instalado Vaultwarden](instalacion-de-vaultwarden-en-docker) la _web vault_ estará alojada en nuestro propio dominio (p.ej. `vault.midominio.com`).

# Extensión del navegador

## Instalación

En mi caso suelo utilizar el navegador [Microsoft Edge](https://www.microsoft.com/en-us/edge) incluído con las últimas versiones de Windows 10/11 y la instalación de la extensión es muy sencilla.

Se puede acceder directamente a la [página web de la extensión](https://microsoftedge.microsoft.com/addons/detail/jbkfoedolllekgbhcbcoahefnbanhhlh) o buscarla desde la [página principal de _addons_](https://microsoftedge.microsoft.com/addons/Microsoft-Edge-Extensions-Home) de este navegador (accesible desde el menú `...` &rarr; `Extensions` del propio navegador).

Una vez en la página de la extensión, se pulsa sobre el botón `Get` y a continuación sobre el botón `Add extension` para descargarla e instalarla:

![Descarga Bitwarden][1]

Una vez instalada, se pulsa sobre el icono similar a una pieza de puzzle para gestionar extensiones del navegador y se activa la visualización de Bitwarden en la barra del navegador:

![Activación extensión][2]

## Configuración

Antes de poder utilizar y configurar Bitwarden es necesario **iniciar sesión** (en la propia nube de Bitwarden o en nuestra propia instalación):

![Inicio de sesión][3]

Antes de empezar a guardar contraseñas, yo suelo configurarla de la siguiente manera accediendo a la pestaña `Settings`:

* SECURITY
  * Vault timeout: `On browser restart`
  * Vault timeout action: `Lock`
* OTHER
  * Options
    * GENERAL
      * Default URI match detection: `Base domain`
      * Clear clipboard: `30 seconds`
      * Copy TOTP automatically: `Disable`
      * Ask to add login: `Disable`
      * Ask to update existing login: `Enable`
      * Show contest menu options: `Enable`
    * DISPLAY
      * Show cards on Tab page: `Enable`
      * Show identities on Tab page: `Enable`
      * Show website icons: `Enable`
      * Show badge counter: `Enable`
      * Theme: `Default`
    * AUTOFILL
      * Auto-fill on page load: `Disable`

Además, también accedo a la pestaña `Generator` para configurar las características de las contraseñas generadas:

* What would you like to generate: `Password`
* Password type: `Password`
* Length: `25`
* A-Z: `Enabled`
* a-z: `Enabled``
* 0-9: `Enabled`
* !@#$%^&*: `Enabled`
* Minimum numbers: 1
* Minimum special: 1
* Avoid ambiguous characters: `Enabled`

## Añadir contraseñas

Para añadir contraseñas únícamente hay que seguir el enlace `Add a login` disponible en la pestaña `Tab` o pulsando el símbolo `+` de la parte superior derecha.

Por defecto, Bitwarden rellenará los campos `Name` y `URI` utilizando la información de la página web actual.

Por ejemplo, si estuviésemos en la [página del reproductor web de Spotify](https://open.spotify.com/) aparecería la siguiente pantalla:

![Creacion de contraseña][4]

donde, como mínimo, deberíamos:

* escribir el **username** (o generar uno aleatorio)
* generar un **password** y seleccionarlo
* grabar la contraseña

> **Nota**: es posible añadir notas a la contraseña o añadir más de una URL si el servicio o página web tiene diferentes dominios.

## Uso de contraseña

Para introducir las contraseñas de forma automática en una determinada página web, se deberá desbloquear el _vault_ usando la **contraseña maestra** y, a continuación, seleccionar la entrada correspondiente para que Bitwarden rellene los campos del formulario de _login_:

![Uso de contraseña][5]

También es posible copiar el usuario, la contraseña o el código TOTP, si existe, pulsando en los iconos correspondientes (persona, llave y reloj).

[1]: /assets/img/blog/2023-01-27_image_1.png "Descarga Bitwarden"
[2]: /assets/img/blog/2023-01-27_image_2.png "Activación extensión"
[3]: /assets/img/blog/2023-01-27_image_3.png "Inicio de sesión"
[4]: /assets/img/blog/2023-01-27_image_4.png "Creación de contraseña"
[5]: /assets/img/blog/2023-01-27_image_5.png "Uso de contraseña"
