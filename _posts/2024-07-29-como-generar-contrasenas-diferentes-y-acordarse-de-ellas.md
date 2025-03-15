---
layout : post
blog-width: true
title: '¿Cómo generar contraseñas diferentes y acordarse de ellas?'
date: '2024-07-29 22:39:19'
last-updated: '2024-07-29 22:39:19'
published: true
tags:
- Seguridad
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2024-07-29_cover.png"
thumbnail-img: ""
---

Seguramente has escuchado decir alguna vez que "**nunca hay que usar una misma contraseña en más de un sitio web**". Pero, ¿te has parado a pensar el porqué de esta afirmación?

En este pequeño artículo, que he escrito para poder referenciarlo cada vez que alguien me dice que usa las mismas contraseñas, intentaré explicarte el porqué y qué puedes hacer para cumplirlo.

# Introducción

Usar la misma contraseña en más de un sitio web puede parecer cómodo (sobretodo si tu memoria no es muy buena) pero representa un riesgo significativo para tu seguridad en línea.

La razón principal por la que no debes hacerlo es el **Credential Stuffing** usando **credenciales robadas**. Se trata de un tipo de ataque en el que los ciberdelincuentes recopilan credenciales de acceso (generalmente listas de nombres de usuario o direcciones de correo electrónico junto con las contraseñas corespondientes) aprovechándose de las vulnerabilidades o la mala configuración de un sitio web.

Estas credenciales robadas pueden estar en texto plano (p.ej. `Password`) o codificadas mediante alguna función tipo _hash_ (p.ej. `e7cf3ef4f17c3999a94f2c6f612e8a888e5b1026878e4e19398b23bd38ec221a` sería el SHA256 de la contraseña anterior).

En el primer caso, los ciberdelincuentes ya dispondrían de una contraseña válida, lista para ser usada. En el segundo caso deberían buscar ese _string_ en ficheros diccionario que contienen millones de _hashes_ para averiguar cuál es la contraseña original.

¿Sabrías decirme ya cuál es el problema de usar una misma contraseña en más de un sitio web? ¿Aún no? No te preocupes, te voy a dar un ejemplo que seguramente aclarará tus dudas.

## Tu cuenta de Google

Imagina que tienes una cuenta de Google que es la dirección de correo electrónico de tu identidad digital online (p.ej. `tunombreyapellidos@gmail.com`).

Como habías escuchado alguna vez que "**las contraseñas tienen que ser difíciles y no estar en diccionarios**", decidiste utilizar `P@l@n24wrt65` como contraseña de esta cuenta (que según [Password Strength Testing Tool](https://bitwarden.com/password-strength/){:target="_blank"} se tardaría en _crackear_ unos 4 meses).

Ahora bien, como tienes poca memoria y te cuesta recordar las contraseñas, también decidiste usar la misma contraseña cuando te registraste en la tienda _online_ "La Huerta de mi Abuela" donde compras frutas y verduras a buen precio. Obviamente, usando tu dirección de correo electrónico `tunombreyapellidos@gmail.com` para realizar el registro.

¿Ves ahora el problema? ¿No? Sigo con el ejemplo ;-)

Resulta que esta web ha sido comprometida recientemente porque está basada en una versión de [WooCommerce vulnerable a ataques XSS](https://developer.woocommerce.com/2024/06/10/developer-advisory-xss-vulnerability-8-8-0/){:target="_blank"} que no ha sido actualizada a tiempo. Los atacantes han conseguido acceder al contenido de la base de datos de usuarios y contraseñas que, para más inri, no estaba cifrado.

El resultado de este ataque ha sido un fichero de texto, compartido en la _Dark Web_, que contiene las credenciales de los miles de clientes de la frutería.

Entre esos miles de direcciones de correo electrónico y contraseñas, que seguramente acaben registradas en [';--have i been pwned?](https://haveibeenpwned.com/){:target="_blank"} en poco tiempo, también está la tuya:


```
[...]
tunombreyapellidos@gmail.com,P@l@n24wrt65
[...]
```

¿Ves ahora el peligro? Supongo que sí, ¿verdad? Imagina que los ciberdelincuentes deciden analizar este fichero para encontrar direcciones de Gmail (`xxxxxxx@gmail.com`). Dado que un gran porcentaje de personas utiliza Gmail como correo electrónico personal, lo más probable es que encuentren muchísimas y, entre ellas, la tuya.

¿Qué crees que será lo primero que hagan cuando encuentren esta información? ¡Correcto, has acertado! Intentarán acceder a tu cuenta de [Gmail](https://www.gmail.com/){:target="_blank"} con el usuario `nombreyapellidos@gmail.com` y la contraseña `P@l@n24wrt65`.

¡Y esa es la contraseña correcta, porque has usado la misma en Gmail y en la frutería!

{: .box-note}
Si no tienes activada la [**verificación en 2 pasos**](https://myaccount.google.com/security){:target="_blank"} (aplicación de autenticación, teléfono, etc.) para validar los inicios de sesión, tu cuenta habrá caído en manos de los ciberdelincuentes.

A partir de aquí imagina lo que pueden llegar a hacer:

* cambiar la contraseña para evitar que tú puedas volver a entrar
* buscar información confidencial entre tus correos de Gmail y tus documentos de Google Drive
* intentar acceder a otros servicios que puedan descubrir, recuperando y cambiando la contraseña ya que ahora tienen el control de tu dirección de correo electrónico
* etc.

# Entonces, ¿qué hago?

Lo ideal, tal como se ha comentado al principio, es **usar contraseñas diferentes en cada servicio web** y que estas contraseñas sean lo suficientemente difíciles para que no se puedan obtener a partir de diccionarios (por ejemplo una combinación aleatorias de números, letras y símbolos).

{: .box-note}
Además, es muy recomendable [**activar el 2FA**](activar-2fa-de-forma-segura) en todos aquellos servicios y web que lo permitan. Si utilizas una aplicación para generar códigos TOTP, debes utilizar una que te permita realizar copias de seguridad de los mismos para evitar quedarte sin acceso al servicio por haberlos perdido.

En este momento, estarás pensando en la gran cantidad de servicios web en los que estás registrado, algunos usados a diario y otros de forma puntual. Y seguramente también estés pensando en la imposibilidad de acordarse de todos ellos y, mucho menos de una contraseña diferente para cada uno, con la memoria que tienes ;-)

Tienes razón, si no tienes una memoria prodigiosa para recordar todas las contraseñas, una solución muy recomendable es utilizar un **gestor de contraseñas** ([KeePassXC](https://keepassxc.org/){:target="_blank"}, [Bitwarden](https://bitwarden.com/){:target="_blank"}, etc.) para almacenarlas de forma segura. Si usamos uno de estos programas, únicamente necesitaremos recordar una **contraseña maestra** para acceder al resto de contraseñas que hayamos guardado en él.

Uno de los gestores de contraseñas más conocido, **Bitwarden**, se puede [configurar fácilmente en el navegador](instalar-y-configurar-bitwarden-en-el-navegador) para ayudarnos a generar y guardar contraseñas completamente aleatorias para cada servicio web.

Bitwarden dispone un plan gratuito para uso personal, un plan para compartir contraseñas entre la familia (6 usuarios por $40 al año), etc. Es una buena solución si no quieres complicarte la vida. Además, [según explican en su FAQ](https://bitwarden.com/es-la/help/security-faqs/){:target="_blank"}, las contraseñas se almacen cifradas mediante la contraseña maestra que únicamente tendréis vosotros.

{: .box-note}
También es posible [instalar Vaultwarden](instalacion-de-vaultwarden-en-docker) en tu propio servidor. Se trata de una implementación no oficial que permite compartir contraseñas entre familiares sin necesidad de pagar una cuota anual. Esta opción no está pensada para todos los públicos ;-)

# ¿Y no puedo guardarlas yo mismo?

Si no quieres utilizar un gestor de contraseñas por el motivo que sea, pero quieres incrementar tu seguridad en línea **dejando de usar la misma contraseña en diferentes servicios web** solo hay dos opciones:

* tener una memoria prodigiosa para acordarse de cada una de ellas ;-)
* utilizar un **algoritmo personal** para generar las contraseñas

¿Qué es este algoritmo personal? Es un método, basado en patrones, mediante el cual podrás **generar una contraseña** suficientemente **larga, compleja y diferente** para cada servicio web.

Este **patrón**, que tendrás que inventarte sin copiarlo de nadie, te permitirá construir una contraseña a partir de bloques de información fija y bloques de información variable en función del servicio.

Los bloques de información pueden estar formados por infinidad de elementos, todos aquellos que se te ocurran, como por ejemplo:

* Palabras ofuscadas mediante:
  * Cambio de letras por números (p.ej. la letra `a` por una `@`)
  * Eliminación de vocales o consonantes
  * Uso de carácteres pares o impares
  * etc.
* Símbolos de separación de bloques
* Nombre de la web ofuscada
* Números (o longitud de alguna palabra)
* Símbolos finales
* etc.

{: .box-note}
Como ves, el método de generación de **tus contraseñas** tiene que ser algo personal, que únicamente conozcas tú y que te sea fácil de recordar. Hay tantos algoritmos personales como personas existen en el mundo.

Un ejemplo de patrón podría ser el siguiente (y no, no lo uses tú, que ahora está publicado en Internet ;-):

* **Palabra** en la que habremos cambiado las vocales por números; la primera letra en mayúscula
* **Símbolo de separación** `@`
* **_Nombre de la web_** eliminando las vocales; la última letra en mayúscula
* **Símbolo de separación** `@`
* **_Longitud_** del nombre de la web, completando con ceros para tener dos cifras
* **Símbolo final** `$`

Usando este patrón se obtienen contraseñas que tienen letras (mayúsculas y minúsculas), números y símbolos. Además, al realizar cambios en las palabras, se consigue evitar encontrarlas en muchos de los diccionarios _online_ que utilizan los ciberdelincuentes.

Por ejemplo, si uso la palabra `Cebra`, ¿sabrías decirme qué contraseña se generaría para acceder a [https://www.elperiodico.com/](https://www.elperiodico.com/){:target="_blank"}? Te dejo unos segundos para pensar ...

Efectivamente, la contraseña sería `C3br4@lprdC@11$`, que tardaría centenares de años en ser _crackeada_ según [Password Strength Testing Tool](https://bitwarden.com/password-strength/){:target="_blank"}:

* Palabra: `Cebra` &rarr; `C3br4`
* Símbolo de separación: `@`
* Nombre de la web: `elperiodico` &rarr; `lprdC`
* Símbolo de separación: `@`
* Longitud del nombre de la web: `11`
* Símbolo final: `$`

Y ahora, ¿te animas a diseñar tu propio algoritmo de generación de contraseñas? ;-)

### Historial de cambios

* **2024-07-29**: Documento inicial
* **2024-07-30**: Revisión y reorganización
