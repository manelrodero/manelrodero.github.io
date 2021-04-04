---
layout : post
blog-width: true
title: 'Borrar un disco de forma segura usando BitLocker'
date: '2021-04-04 12:55:48'
published: true
tags:
- Seguridad
author:
  display_name: Manel Rodero
---

Una forma rápida y segura de borrar cualquier disco HDD, USB o SSD es utilizar **BitLocker**.

La idea es la siguiente:

1. Se encripta el disco usando BitLocker
2. Se formatea el disco
3. Se vuelve a encriptar el disco. De esta manera, la clave de encriptación del paso 1 será sobreescrita, haciendo imposible la recuperación de los datos encriptados (el disco tendrá datos aleatorios sin sentido)
4. Se formatea de nuevo el disco para dejarlo listo para usarlo de nuevo

# Paso 1 (Encriptación)

* Conectar el disco al sistema y esperar a que éste sea detectado y Windows le asigne una letra (en este ejemplo es la **G:**)
* Buscar **BitLocker** en el menú de inicio y ejecutarlo (también es posible hacerlo desde `Control Panel\System and Security\BitLocker Drive Encryption`)
* Localizar el disco y seleccionar la opción **Turn on BitLocker**

![BitLocker][1]

* Introducir un [password aleatorio](https://passwordsgenerator.net/), ya que no hace falta recordarlo

![Password aleatorio][2]

* Guardar el fichero de texto con la clave de recuperación (`BitLocker Recovery Key <GUID>.TXT`) en el escritorio y después borrarlo

![Fichero recovery key][3]

* Seleccionar **Encrypt entire drive** para asegurarse que se encriptan también los ficheros borrados

![Encriptar disco entero][4]

* Seleccionar **Compatible mode** ya que después se formateará y no es importante

![Modo compatible][5]

* Pulsar el botón `Start encrypting` para que comience el proceso. Es algo lento y puede tardar un buen rato (2h18m para un disco mecánico de 300GB)

![Encriptando][6]

Al finalizar este proceso el disco estará encriptado con BitLocker:

![BitLocker activado][7]

# Paso 2 (Formateo)

* Abrir el Explorador de archivos de Windows
* Localizar el disco y seleccionar la opción **Format**
* Hacer un formateo rápido para "eliminar" la encriptación

# Paso 3 (Encriptación)

Realizar el mismo procedimiento que en el paso 1 usando un password aleatorio **diferente**.

![Passwordgenerator.net][8]

Ahora se puede elegir **Encrypt used disk space only** ya que únicamente hay que sobreescribir la clave de encriptación que aún está en el disco.

Este proceso finalizará en pocos segundos.

# Paso 4 (Formateo)

Realizar el mismo procedimiento que en el paso 2. Una vez formateado, el disco se habrá sobreescrito con datos encriptados y la clave de encriptación original se ha destruido.

**Cualquier dato del disco es irrecuperable**, ya que son **datos aleatorios**. El disco se puede reusar o entregar a terceros sin preocupaciones.

<p></p>

[1]: /assets/img/blog/2021-04-04_image_1.png "BitLocker"
[2]: /assets/img/blog/2021-04-04_image_2.png "Password aleatorio"
[3]: /assets/img/blog/2021-04-04_image_3.png "Fichero recovery key"
[4]: /assets/img/blog/2021-04-04_image_4.png "Encriptar disco entero"
[5]: /assets/img/blog/2021-04-04_image_5.png "Modo compatible"
[6]: /assets/img/blog/2021-04-04_image_6.png "Encriptando"
[7]: /assets/img/blog/2021-04-04_image_7.png "BitLocker activado"
[8]: /assets/img/blog/2021-04-04_image_8.png "Passwordgnerator.net"