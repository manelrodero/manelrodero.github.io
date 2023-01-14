---
layout : post
blog-width: true
title: 'Autenticación SSH usando una clave privada'
date: '2023-01-14 13:12:29'
published: true
tags:
- HomeLab
author:
  display_name: Manel Rodero
---

Hasta ahora siempre he usado usuario y contraseña para autenticarme en el servidor SSH de la Raspberry Pi 4 de mi _HomeLab_.

Ahora ha llegado el momento de utilizar autenticación mediante clave pública/privada para facilitar el acceso (siempre es bueno guardar la contraseña en un gestor de contraseñas para no olvidarla por no usarla a partir de ahora).

# Crear claves pública/privada

Para crear un par de claves pública/privada hay que ejecutar el comando `ssh-keygen` incluído con el cliente de SSH.

En este caso he ejecutado el comando desde mi ordenador personal con sistema operativo Windows para generar una clave `ECDSA` de `521 bits`

```
ssh-keygen -t ecdsa -b 521 -C "pi@raspberry"
```

Se puede elegir una **frase** como **contraseña** de la clave privada que se generará tal como se puede ver en el siguiente ejemplo:

```
Generating public/private ecdsa key pair.
Enter file in which to save the key (C:\Users\<user>/.ssh/id_ecdsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in C:\Users\<user>/.ssh/id_ecdsa
Your public key has been saved in C:\Users\<user>/.ssh/id_ecdsa.pub
The key fingerprint is:
SHA256:d13b62Z4dXlYTSHG97sPnQBSgar5mNZy3tFuYg2s35o pi@raspberry
The key's randomart image is:
+---[ECDSA 521]---+
|          .ooo ..|
|         .. .....|
|        .. .  ..+|
|       .  . .. .*|
|      o S . ...++|
|     o   +..  o+*|
|      = ..o.  oo*|
|     = =.o=o ..* |
|    . +.oE+o  +.o|
+----[SHA256]-----+
```

# Copiar clave pública al servidor

La forma más sencilla de hacerlo sería usar el comando `ssh-copy-id` pero la implementación de **OpenSSH** en Windows no lo incorpora tal como se muestra en el siguiente listado del directorio `C:\Windows\System32\OpenSSH`:

```
06/05/2022  15:15           320.512 scp.exe
06/05/2022  15:15           398.848 sftp.exe
06/05/2022  15:15         1.073.152 ssh.exe
06/05/2022  15:15           506.880 ssh-add.exe
06/05/2022  15:15           393.216 ssh-agent.exe
06/05/2022  15:15           720.896 ssh-keygen.exe
06/05/2022  15:15           572.416 ssh-keyscan.exe
```

El método manual para copiar la clave pública consiste en añadir el contenido del fichero `id_ecdsa.pub` al fichero `~/.ssh/authorized_keys` del servidor.

```
type "%userprofile%\.ssh\id_ecdsa.pub"
```

En este ejemplo, se copia el siguiente texto que incluye el algoritmo utilizado, la clave pública y la identificación de la misma:

```
ecdsa-sha2-nistp521 AAAAE2VjZHNhLXNoYTItbmlzdHA1MjEAAAAIbmlzdHA1MjEAAACFBAGV91+ldUiuM8pZRsC2W1OAjRv+Ttm1pabs2TF3Y75iVTukxytHO1QFH+nJ6T6/wNq80P87II95Y+mwpPi8b+Jt4AFujHmNKk4vHHDpCCRh/SPbLxPI8VqO9gFgq07P9KTIFvUiJPg65kgy5a9b4DaWmkb5baVBn701lCSqhy7gtHDXWg== pi@raspberry
```

Después nos conectamos al servidor (todavía mediante usuario y contraseña), editamos el fichero `authorized_keys` y pegamos el texto anterior.

Una vez hecho ésto, se desconecta la sesión del servidor y, si todo va bien, deberíamos poder volver a conectarnos mediante un simple `ssh pi@raspberry`.

```
Linux raspberry 5.10.103-v8+ #1529 SMP PREEMPT Tue Mar 8 12:26:46 GMT 2022 aarch64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sat Jan 14 18:01:55 2023 from 10.10.1.2
```

A partir de aquí, se puede complicar todo lo que uno quiera. Solo hay que leer los artículos de referencia que se indican a continuación ;-)


# Referencias

* [SSH Key Best Practices](https://dev.to/paulmicheli/ssh-key-best-practices-2cb7)
* [SSH Best Practices using Certificates, 2FA and Bastions](https://goteleport.com/blog/how-to-ssh-properly/)
