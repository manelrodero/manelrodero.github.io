---
layout : post
blog-width: true
title: 'Activar 2FA de forma segura'
date: '2022-04-02 17:19:25'
published: true
tags:
- Seguridad
author:
  display_name: Manel Rodero
---

La activación de un segundo factor de autenticación (_two-factor authentication_ o **2FA**) permite añadir un nivel extra de  protección a nuestras cuentas de usuario de servicios _online_.

Cuando se ha activado 2FA, para acceder a nuestra cuenta de usuario se necesita, además del **nombre de usuario** (_username_) y la **contraseña** (_password_), una **información adicional** proveniente de alguna de estas categorías:

* **Algo que conoces**: un PIN (_personal identification number_), otra contraseña, la respuesta a una "pregunta secreta", un patrón de teclas específico, etc.
* **Algo que tienes**: una tarjeta inteligente, un teléfono, un pequeño token hardware, etc.
* **Algo que eres**: el patrón biométrico de la huella digital, un escaneo del iris del ojo, una impresión de voz, etc.

Uno de los métodos 2FA más básicos se basa en enviar un SMS al teléfono del usuario. Después de introducir un nombre de usuario y una contraseña válidos en el servicio _online_, se envía al usuario un código de un único uso ([**OTP**](https://en.wikipedia.org/wiki/One-time_password), _one-time password_) que éste deberá introducir para completar el acceso.

{: .box-warning}
**Nota**: [El uso de SMS como 2FA no se considera seguro](https://www.contalks.com/talks/280/why-using-sms-in-the-authentication-chain-is-risky-appsec-usa-2016) debido a la existencia de técnicas mediante las cuales un ciberdelincuente puede suplantar el teléfono del usuario para recibir el código OTP (por ejemplo el [_SIM swapping_](https://privacypros.io/u2f/sim-swapping/)).

# Software tokens

Una de las formas más populares de 2FA es la generación por software de códigos de un único uso basados en tiempo ([**TOTP**](https://en.wikipedia.org/wiki/Time-based_one-time_password), _time-based one-time password_).

Para usarlos, es necesario descargar e instalar una aplicación 2FA en el teléfono móvil o en el ordenador. Esta aplicación genera un código temporal (habitualmente cada 30 segundos) que será necesario introducir, después del nombre de usuario y la contraseña, para acceder a un determinado servicio _online_:

![Ejemplo de aplicación 2FA. Source: https://grantwinney.com/][5]

Estas aplicaciones generan estos códigos temporales mediante un algoritmo que utiliza una **clave secreta** que proporciona cada servicio, normalmente mediante un **código QR**, en el momento de activar 2FA en el mismo.

# Activación de 2FA

La mayoría de servicios _online_ que se preocupan por la seguridad permiten activar 2FA o 2SV (_two-step verification_) desde alguna opción de la configuración de la cuenta relacionada con la seguridad.

## Opciones de seguridad

En el siguiente ejemplo, esta opción está en la sección `Your profile` &rarr; `Security settings` &rarr; `Two-Factor Authentication`. Pulsaremos `Click here to Enable` para comenzar el proceso:

![Activación 2FA][1]

## Método 2FA

Si el servicio _online_ soporta diferentes tipos de 2FA, intentaremos evitar el uso del teléfono/SMS y elegiremos la opción para generar códigos TOTP mediante una aplicación:

![Elección de TOTP][2]

{: .box-note}
**Nota**: Aunque el sitio web indique [Google Authenticator](https://googleauthenticator.net/), se puede utilizar cualquier aplicación que permita generar códigos TOTP: [Microsoft Authenticator](https://www.microsoft.com/es-es/security/mobile-authenticator-app), [Authy](https://authy.com/), [andOTP](https://github.com/andOTP/andOTP), etc.

## Código QR y clave secreta

A continuación, el servicio suele mostrar un **código QR** que se puede escanear mediante la aplicación 2FA para añadir el servicio a la misma. A veces, también se proporciona la **clave secreta** en forma de texto para poder añadir el servicio a la aplicación 2FA de forma manual:

![QR y clave secreta][3]

{: .box-warning}
**Nota**: Hay quien argumenta que [no se deben escanear los códigos QR 2FA](https://medium.com/crypto-punks/why-you-shouldnt-scan-two-factor-authentication-qr-codes-e2a44876a524) debido a que éstos incluyen más información de la necesaria para generar los códigos TOTP (por ejemplo, la dirección de correo electrónico o la URL del servicio).

Si pierdes o te roban el teléfono, y la aplicación 2FA no realiza copias de seguridad de la información, podríamos perder el acceso a nuestros servicios _online_ al no poder introducir el segundo factor de autenticación. Por ese motivo, se recomienda **guardar en un lugar seguro** los **códigos QR** y la **clave secreta** para poder volver a introducirlos en la aplicación si fuese necesario.

{: .box-warning}
**Nota**: Este lugar seguro [no debería ser nuestro gestor de contraseñas habitual](https://blog.securityevaluators.com/psa-dont-store-2fa-codes-in-password-managers-77d92608b062) ya que, si un atacante consigue obtener acceso al mismo, tendría todo lo necesario para acceder a nuestros servicios _online_. Por este motivo, hay quien [imprime los códigos QR](https://www.cloudsavvyit.com/8387/the-trick-to-almost-never-loosing-2fa-mfa-access/) para guardarlos fuera del ordenador.

## Guardado seguro de la información 2FA

Una solución al problema de guardar la información 2FA de forma segura podría ser la siguiente:

* Gestor de contraseñas principal (por ejemplo [Bitwarden](https://bitwarden.com/)) para guardar:
  * nombre del servicio y URL del mismo
  * nombre de usuario
  * contraseña (**generada de forma aleatoria**)
  * notas sobre el servicio
  * etc.

![Bitwarden][8]

* Gestor de contraseñas **local** (por ejemplo [KeePassXC](https://keepassxc.org/)) para guardar **únicamente**:
  * nombre del servicio
  * imagen adjunta del código QR (en formato `png` o `jpg`)
  * clave secreta
  * _backup codes_ (si se proporcionan)

![KeePassXC][9]

{: .box-warning}
**Nota**: Si se necesita más seguridad, la base de datos de este gestor de contraseñas local se puede almacenar en una unidad encriptada (por ejemplo usando [Cryptomator](https://cryptomator.org/)).

## Añadir el servicio a la aplicación 2FA

A continuación, se añade el servicio a la aplicación 2FA de **forma manual** introduciendo la **clave secreta** que se ha guardado en el gestor de contraseñas local. De esta forma, si todo funciona correctamente, se garantiza que la información guardada es correcta para una posible recuperación en el futuro.

Si el servicio no proporciona la clave secreta, se puede utilizar una aplicación para escanear códigos QR (por ejemplo, [Barcode Scanner](https://play.google.com/store/apps/details/Barcode_Scanner?id=com.google.zxing.client.android&gl=US)) para obtenerla a partir de la [URI](https://github.com/google/google-authenticator/wiki/Key-Uri-Format) `otpauth://totp` tal como se muestra en la siguiente imagen:

![Obtención de la clave secreta a partir del QR][6]

## Verificación del 2FA

Finalmente, se requerirá introducir el código TOTP obtenido mediante la aplicación 2FA para que el servicio lo valide:

![Introducción del código OT][7]

Si la validación es correcta, el servicio quedará configurado para usar 2FA a partir de ese instante. Al final del proceso, algunos servicios también proporcionan uno o más _backup codes_ de un solo uso para usarlos cuando no se puede utilizar la aplicación 2FA:

![Backup codes][4]

{: .box-note}
**Nota**: Es muy aconsejable guardar los _backup codes_ en el mismo **gestor de contraseñas local** que el código QR y la clave secreta.

[1]: /assets/img/blog/2022-04-02_image_1.png "Activación 2FA"
[2]: /assets/img/blog/2022-04-02_image_2.png "Elección de TOTP"
[3]: /assets/img/blog/2022-04-02_image_3.png "QR y clave secreta"
[4]: /assets/img/blog/2022-04-02_image_4.png "Backup codes"
[5]: /assets/img/blog/2022-04-02_image_5.png "Ejemplo de aplicación 2FA. Source: https://grantwinney.com/"
[6]: /assets/img/blog/2022-04-02_image_6.png "Obtención de la clave secreta a partir del QR"
[7]: /assets/img/blog/2022-04-02_image_7.png "Introducción del código OTP"
[8]: /assets/img/blog/2022-04-02_image_8.png "Bitwarden"
[9]: /assets/img/blog/2022-04-02_image_9.png "KeepassXC"