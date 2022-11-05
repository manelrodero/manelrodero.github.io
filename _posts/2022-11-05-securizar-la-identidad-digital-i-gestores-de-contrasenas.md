---
layout : post
blog-width: true
title: 'Securizar la identidad digital (I): gestores de contraseñas'
date: '2022-11-05 12:02:19'
published: true
tags:
- Seguridad
author:
  display_name: Manel Rodero
---

Este _post_ es el primero de una serie de artículos donde explicaré como **aumentar la seguridad de nuestra identidad digital** en Internet, una de las preguntas que más a menudo suelen hacerme, usando:

1. Gestores de contraseñas
2. Gestores de _tokens_ 2FA
3. Gestores de _alias_ de correo electrónico

# ¿Qué es la identidad digital?

Nuestra identidad digital es el conjunto de información sobre nosotros existente en los diferentes servicios web de Internet que utilizamos a diario.

Para utilizar estos servicios utilizamos una cuenta de usuario en la que se guarda **información personal** y de **comportamiento** que, al combinarse, definen un perfil muy preciso de cada uno de nosotros.

Si un ciberdelicuente tuviese acceso a nuestras cuentas de usuario podría obtener gran cantidad de datos sobre nosotros:

* Personales (nombre, dirección, teléfono, ...)
* Económicos (salario, tarjetas de crédito, ...)
* Médicos (historial, enfermedades, ...)
* Comportamiento (búsquedas, música, hobbies, ...)
* Sociales (amigos, afinidad política, ...)
* Etc.

Por esta motivo es muy importante **proteger** estas cuentas de usuario mediante todas las opciones que tenga disponible cada servicio web: **contraseñas robustas**, **autenticación de doble factor** (2FA), **bloqueo de las cuentas**, etc.

# Contraseñas

Seguramente habremos escuchado en alguna ocasión alguna de las siguientes **recomendaciones básicas** sobre el uso de las contraseñas para que éstas sean seguras:

* Que **no sean sencillas** (por ejemplo `123456`, `qwerty`, etc.)
* Que **no sean fácilmente deducibles** (por ejemplo la fecha de nacimiento, que sigan un patrón determinado, etc.)
* Que tengan una **longitud mínima** de 10 carácteres, idealmente más
* Que sean **aleatorias** y contengan **mayúsculas**, **minúsculas**, **números** y **símbolos** (por ejemplo `Mbn$76%dvb98z#234g`)
* Que sean **diferentes** para cada servicio o sitio web
* Cambiarlas cada cierto tiempo
* **No compartirlas** con nadie
* **No anotarlas** en una libreta, _post-it_, etc.
* Que las respuestas a las preguntas de seguridad sean tan seguras como las contraseñas
* Etc.

Como se habrá deducido, cumplir con éstas y otras recomendaciones sobre la seguridad de las contraseñas es prácticamente imposible si dependemos únicamente de nuestra memoria.

Por ese motivo se recomienda utilizar un **programa para gestionar y almacenar las contraseñas** de forma segura y cumpliendo las recomendaciones anteriores. Este programa se encargará de generar contraseñas aleatorias y difíciles para cada servicio web que usemos.

Al usar contraseñas aleatorias y largas es menos probable que alguien consiga adivinarlas y acceder a nuestras cuentas. Además, al usar una contraseña diferente para cada servicio web, se evita que un ciberdelincuente utilice la contraseña robada en una brecha de seguridad de un servicio web para acceder a otro.

> **Nota**: Puedes comprobar si tu dirección de correo electrónico o teléfono han sido expuestos en alguna de estas brechas de seguridad utilizando el servicio [**';--have i been pwned?**](https://haveibeenpwned.com/).

# Gestores de contraseñas

La mayoría de navegadores incorporan algún método para **almacenar** las contraseñas y **recordarlas** cuando se va a iniciar la sesión en un sitio web.

Ahora bien, estos gestores son bastante básicos y no tienen la seguridad de los programas especializados, por lo que se aconseja **deshabilitarlos** y **borrar** todas las contraseñas que tengamos almacenadas en ellos:

* [Microsoft Edge](https://support.microsoft.com/es-es/microsoft-edge/guardar-o-dejar-de-recordar-contrase%C3%B1as-en-microsoft-edge-b4beecb0-f2a8-1ca0-f26f-9ec247a3f336)
* [Google Chrome](https://support.google.com/chrome/answer/95606?hl=es)
* [Mozilla Firefox](https://support.mozilla.org/es/kb/administrador-de-contrasenas-recordar-borrar-cambiar-importar-contrase%C3%B1as-firefox)

A día de hoy, hay decenas de programas que permiten gestionar las contraseñas de manera segura, algunos de ellos gratuitos y otros de pago. Algunos programas de pago tienen versiones gratuitas con menos funcionalidades.

La mayoría de estos programas permiten generar contraseñas aleatorias y fuertes para cada servicio web o cuenta, evitando así el repetirlas o que sean fáciles de adivinar.

Lo más habitual suele ser instalarlos en el navegador, pero también es posible que tengan una aplicación de escritorio o para móvil. En ese caso, las contraseñas **cifradas** se sincronizan entre los diferentes dispositivos usando la nube del fabricante.

Algunos de los gestores de contraseñas más utilizados (sin ningún orden de preferencia) son los siguientes:

| Gestor de contraseñas | Gratuito | De pago |
|-----|:-----:|:-----:|
| [1Password](https://1password.com/es/sign-up/) | | x |
| [LastPass](https://www.lastpass.com/es/pricing) | x | x |
| [Dashlane](https://www.dashlane.com/es/pricing) | | x |
| [KeePass](https://keepass.info/) | x | |
| [KeePassXC](https://keepassxc.org/) | x | |
| [Enpass](https://www.enpass.io/pricing/) | | x |
| [Keeper](https://www.keepersecurity.com/es_ES/pricing/personal-and-family.html) | | x |
| [Bitwarden](https://bitwarden.com/) | x | x |
| [PasswordSafe](https://pwsafe.org/) | x | |
| [Roboform](https://www.roboform.com/es/everywhere) | x | x |

# Bitwarden

En mi caso, he usado **KeePass** (y últimamente **KeePassXC**) desde hace más de 10 años, para gestionar y almacenar mis contraseñas de manera local en mi ordenador personal.

Con estos programas, la sincronización de las contraseñas entre diferentes dispositivos (por ejemplo, un portátil o un teléfono móvil) es algo que tienes que hacer tú mismo, ya sea copiando las bases de datos de contraseñas de forma manual o usando Google Drive, Microsoft OneDrive o similar.

Desde hace un par de años estoy utilizando **Bitwarden**, uno de los mejores gestores de contraseñas de código abierto que existen en la actualidad, mediante el **complemento** del navegador, la aplicación de **escritorio**  o la aplicación para **móvil**.

> **Nota**: A la hora de decidirse por uno u otro, es recomendable leer algún análisis comparativo como éste de [**PCWorld**](https://www.pcworld.es/mejores-productos/seguridad/gestores-contrasenas-3680297/) y, si es posible, probarlos nosotros mismos durante un tiempo.

## Cuenta gratuita

La forma más sencilla para comenzar a utilizar Bitwarden es crear una **cuenta (gratuita** para siempre) desde su página web [https://bitwarden.com/](https://bitwarden.com/).

Esta opción creará un _vault_ (almacén de contraseñas en los servidores de Bitwarden) que permite utilizar todas las [funcionalidades principales](https://bitwarden.com/help/) del producto:

* Código abierto
* Cifrado de los datos
* Dispositivos ilimitados + sincronización
* Navegador, móvil, aplicaciones de escritorio
* Contraseñas ilimitadas
* Almacenaje de notas, tarjetas de crédito y contraseñas
* Compartición gratuita entre 2 usuarios
* Bitwarden Send
* Generador de contraseñas
* Integración de alias de correo electrónico
* Inicio de sesión básico en dos pasos
* Exportación encriptada

Para crear la cuenta hay que acceder a la [página de registro](https://vault.bitwarden.com/#/register) y rellenar un formulario con los siguientes datos:

* Dirección de correo electrónico
* Nombre
* **Master Password** (contraseña maestra mediante la cual se cifrarán nuestros datos):
  * ¡Es la única contraseña que deberemos recordar!
  * Es importante **que sea muy segura** (por ejemplo una frase muy larga que contenga mayúsculas, minúsculas, números y símbolos)
  * **¡Es importante recordarla!** Si no es así, será imposible descifrar los datos y habremos perdido todas nuestras contraseñas :-(
* Pista opcional para recordar la contraseña maestra (¡que no sea algo obvio para nadie!)
* Aceptar las condiciones de licencia

Una vez creada, ya se puede iniciar sesión con la dirección de correo electrónico y la contraseña maestra tal como se muestra a continuación:

![Bitwarden Login][1]

> **Nota**: al iniciar sesión por primera vez, aparecerá un mensaje indicando que es necesario verificar el correo electrónico. Después de pulsar el botón `Send email`, nos llegará un correo electrónico con un botón `Verify Email Address Now` que deberemos pulsar para completar la verificación.

A partir de aquí ya se puede utilizar la **interfaz web** para gestionar nuestras contraseñas tal como se explica en la página de ayuda "[Get Started with the Web Vault](https://bitwarden.com/help/getting-started-webvault/)":

![Web Vault][2]

Aunque es posible usar únicamente la interfaz web de Bitwarden, es recomendable [instalar una extensión en el navegador](https://bitwarden.com/help/getting-started-browserext/) para facilitar la generación de contraseñas aleatorias al registrarnos en un sitio web o introducir de forma fácil el usuario y contraseña al iniciar sesión en un servicio web:

![Browser Extension][3]

También es recomendable instalar la [aplicación móvil](https://bitwarden.com/help/getting-started-mobile/) para tener acceso a nuestras contraseñas cuando estamos fuera de casa y no queremos acceder a la interfaz web de Bitwarden:

![Mobile App][5]

Esta aplicación móvil mantiene una copia local de los datos que permite acceder a las contraseñas de forma _offline_ (sin conexión con los servidores de Bitwarden) durante 90 días.

Instalar la [aplicación de escritorio](https://bitwarden.com/help/getting-started-desktop/) es también interesante por el mismo motivo, aunque en este caso únicamente mantiene una copia local de los datos durante 30 días.

![Desktop App][4]

> **Nota**: esta _caché_ de los datos para usarlos de forma _offline_ se actualiza cada vez que usamos la aplicación y la desbloqueamos con la contraseña maestra.

# ¿Y ahora qué hago?

Ahora deberías estar pensando seriamente en comenzar a utilizar un **gestor de contraseñas** y dejar de utilizar contraseñas repetidas, sencillas o que tienes apuntadas en un papel ;-)

¿Cuál utilizar? El que más te convenza, para gustos no hay nada escrito. Leer análisis comparativos y probarlos uno mismo te ayudarán a elegir el más adecuado a tus necesidades.

 Si te pareció que Bitwarden es una buena opción, puedes buscar en YouTube algún tutorial que lo explique más en detalle para acabar de estar convencido (por ejemplo "[Bitwarden Tutorial For Beginners - How To Use Bitwarden For Beginners](https://youtu.be/etF4rbWk-pg)").

> **Nota**: Recuerda que, cuando comiences a usar un gestor de contraseñas, lo **primero** que deberás hacer es **acceder a todos tus servicios web para cambiar tu contraseña actual** por una aleatoria y más segura generada desde el propio gestor.

[1]: /assets/img/blog/2022-11-05_image_1.png "Bitwarden Login"
[2]: /assets/img/blog/2022-11-05_image_2.png "Web Vault"
[3]: /assets/img/blog/2022-11-05_image_3.png "Browser Extension"
[4]: /assets/img/blog/2022-11-05_image_4.png "Desktop App"
[5]: /assets/img/blog/2022-11-05_image_5.png "Mobile App"
