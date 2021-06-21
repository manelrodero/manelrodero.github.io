---
layout : post
blog-width: true
title: 'Creación de un "Trial Tenant" de Microsoft 365'
date: '2021-06-21 13:24:11'
published: true
tags:
- Microsoft 365
author:
  display_name: Manel Rodero
---

Hace un par de meses expliqué [cómo crear un _tenant_ de desarrollo de Microsoft 365](/blog/creacion-de-un-dev-tenant-de-microsoft-365) para tener una suscripción "**Microsoft 365 E5 Developer (without Windows and Audio Conferencing) Trial**" que se va renovando cada 90 días de forma automática si se utiliza el _tenant_.

Ahora bien, a veces es necesario crear un _tenant_ de usar y tirar para probar alguna funcionalidad nueva, para realizar una presentación o grabar un vídeo, por formación, etc.

En ese caso se puede utilizar la versión de prueba por 90 días de [Microsoft Enterprise Mobility + Security (EMS) E5](https://www.microsoft.com/en-us/security/business/enterprise-mobility-security)

## Activar prueba

Los pasos para obtener un _tenant_ de prueba son los siguientes:

{: .box-note}
**Nota**: Se recomienda realizar este proceso desde una **ventana de incógnito** para evitar interferencias con otras cuentas Microsoft.

1) Acceder a la página [EMS E5 Free Trial](https://go.microsoft.com/fwlink/p/?LinkID=2077039&clcid=0x409&culture=en-us&country=US)

2) Introducir una dirección de correo electrónico y pulsar el botón `Next` para comprobarla

3) Si la dirección de correo electrónico no se había utilizado antes será necesario crear una cuenta pulsando el botón `Set up account` e introducir la siguiente información personal:

{: .box-note}
**Nota**: El teléfono tiene que ser un número real para poder recibir un SMS de confirmación.

```
First name: <>
Middle name (Optional): <>
Last name: <>
Business phone number: <>
Company name: <>
Company size: <>
Country or region: <>
```

4) A continuación se elige la opción para obtener el SMS (p.ej. **Text me**) y se pulsa el botón `Send verification code`. Cuando llegue el SMS con el código de verificación se introduce en la casilla correspondiente y se pulsa el botón `Verify`

5) Elegir un nombre para el dominio **onmicrosoft.com** y comprobar si está disponible mediante el botón `Check availability`

6) Crear la cuenta y la contraseña del administrador del _tenant_ y pulsar el botón `Sign up`:

```
Username: <>
Password: <>
Confirm password: <>
```

7) Confirmar los detalles pulsando sobre el botón `Get Started`:

```
Thanks for signing up for Enterprise Mobility + Security E5
Your username is user@domain.onmicrosoft.com
We've sent a confirmation email to user@domain.com
```

Si todo funciona correctamente, se dispondrá de una suscripción **Enterprise Mobility + Security E5 Trial** válida por 90 días con 250 licencias disponibles para explorar los recursos de [Azure](https://azure.microsoft.com/en-us/).

![Imagen][1]

[1]: /assets/img/blog/2021-06-21_image_1.png "Imagen"
