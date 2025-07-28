---
layout : post
blog-width: true
title: 'Autenticación SSH usando una clave privada'
date: '2023-01-14 13:12:29'
last-updated: '2025-07-27 10:56:30'
published: true
tags:
- HomeLab
author:
  display_name: Manel Rodero
---

Hasta ahora siempre he usado usuario y contraseña para autenticarme en el servidor SSH de la Raspberry Pi 4 de mi _HomeLab_.

Ahora ha llegado el momento de utilizar autenticación mediante clave pública/privada para facilitar el acceso (siempre es bueno guardar la contraseña en un gestor de contraseñas para no olvidarla por no usarla a partir de ahora).

## Crear claves pública/privada

Para crear un par de claves pública/privada hay que ejecutar el comando `ssh-keygen` incluído con el cliente de SSH.

En este caso he ejecutado el comando desde mi ordenador personal con sistema operativo Windows para generar una clave `ECDSA` de `521 bits`

```bash
ssh-keygen -t ecdsa -b 521 -C "%username%@%computername%"
```

Se puede elegir una **frase** como **contraseña** de la clave privada que se generará tal como se puede ver en el siguiente ejemplo:

```plaintext
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

## Copiar clave pública al servidor

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

```cmd
type "%userprofile%\.ssh\id_ecdsa.pub"
```

En este ejemplo, se copia el siguiente texto que incluye el algoritmo utilizado, la clave pública y la identificación de la misma:

```plaintext
ecdsa-sha2-nistp521 AAAAE2VjZHNhLXNoYTItbmlzdHA1MjEAAAAIbmlzdHA1MjEAAACFBAGV91+ldUiuM8pZRsC2W1OAjRv+Ttm1pabs2TF3Y75iVTukxytHO1QFH+nJ6T6/wNq80P87II95Y+mwpPi8b+Jt4AFujHmNKk4vHHDpCCRh/SPbLxPI8VqO9gFgq07P9KTIFvUiJPg65kgy5a9b4DaWmkb5baVBn701lCSqhy7gtHDXWg== username@computername
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

A partir de aquí, se puede complicar todo lo que uno quiera. Solo hay que leer los artículos de referencia indicados al final de este artículo ;-)

# Tipos de claves

## Actuales

| Tipo de clave | Archivo | Seguridad estimada | Rendimiento | Compatibilidad | Comentario |
| --- | --- | --- | --- | --- | --- |
| RSA 3072 | `ssh_host_rsa_key` | Alta (128 bits) | Medio | Muy alta | Buena compatibilidad y seguridad. Aunque 4096 bits es más robusto, 3072 es un buen equilibrio |
| ED25519 | `ssh_host_ed25519_key` | Muy alta (128 bits) | Muy alto | Alta | Muy segura, rápida y moderna. **Recomendadísima por OpenSSH si no se necesita compatibilidad con sistemas muy antiguos** |
| ECDSA 256 | `ssh_host_ecdsa_key` | Alta (128 bits) | Alto | Alta | Rápida y eficiente, pero depende de la confianza en las curvas NIST (algunos expertos son escépticos) |

## Obsoletas

| Tipo de clave | Archivo | Seguridad estimada | Rendimiento | Compatibilidad | Comentario |
| --- | --- | --- | --- | --- | --- |
| RSA < 2048 | `ssh_host_rsa_key` | Baja | Medio | Alta | Consideradas débiles hoy en día. No se recomienda usar claves RSA menores a 2048 bits |
| DSA 1024 | `ssh_host_dsa_key` | Muy baja | Medio | Baja | Obsoleta, insegura, y deshabilitada por defecto desde OpenSSH 7.0. No debe usarse |
| ECDSA (curvas no estándar) | `ssh_host_ecdsa_key` | Variable | Alto | Variable | Riesgos si no se usan curvas bien auditadas. Se recomienda usar solo curvas estándar como `nistp256` |

## Regenerar claves del servidor

A continuación se muestran un par de _scripts_ que permiten regenerar las claves SSH que utilizará un servidor.

El primero de ellos se encarga de **generar** las claves recomendadas `ED25519` y `RSA (3072 bits)` y de **borrar** las claves DSA y ECDSA por ser obsoletas o no recomendadas:

```bash
#!/bin/bash

set -e

echo "🔐 Eliminando claves antiguas DSA y ECDSA si existen..."
rm -f /etc/ssh/ssh_host_dsa_key*
rm -f /etc/ssh/ssh_host_ecdsa_key*

echo "🔑 Regenerando clave ED25519..."
yes | ssh-keygen -t ed25519 -f /etc/ssh/ssh_host_ed25519_key -N ""

echo "🔑 Regenerando clave RSA (3072 bits)..."
yes | ssh-keygen -t rsa -b 3072 -f /etc/ssh/ssh_host_rsa_key -N ""

echo "✅ Claves SSH regeneradas correctamente."
```

El segundo se encarga de modificar el fichero de configuración del servidor SSH `/etc/ssh/sshd_config` para utilizar únicamente las claves anteriores:

```bash
#!/bin/bash

CONFIG="/etc/ssh/sshd_config"
BACKUP="/etc/ssh/sshd_config.bak"

# Crear copia de seguridad
cp "$CONFIG" "$BACKUP"

# Procesar el archivo
awk '
BEGIN { found_pubkey = 0 }
{
    if ($0 ~ /^[# ]*HostKey[ \t]+\/etc\/ssh\/ssh_host_rsa_key/) {
        print "HostKey /etc/ssh/ssh_host_rsa_key"
    } else if ($0 ~ /^[# ]*HostKey[ \t]+\/etc\/ssh\/ssh_host_ed25519_key/) {
        print "HostKey /etc/ssh/ssh_host_ed25519_key"
    } else if ($0 ~ /^[^#]*HostKey[ \t]+\/etc\/ssh\/ssh_host_dsa_key/) {
        print "#" $0
    } else if ($0 ~ /^[^#]*HostKey[ \t]+\/etc\/ssh\/ssh_host_ecdsa_key/) {
        print "#" $0
    } else if ($0 ~ /^PubkeyAcceptedKeyTypes/) {
        print "PubkeyAcceptedKeyTypes ssh-ed25519,ssh-rsa"
        found_pubkey = 1
    } else {
        print $0
    }
}
END {
    if (found_pubkey == 0) {
        print ""
        print "PubkeyAcceptedKeyTypes ssh-ed25519,ssh-rsa"
    }
}
' "$BACKUP" > "$CONFIG"

echo "✅ Archivo actualizado: $CONFIG"
```

> Después de haber generado las claves y modificado el fichero de configuración, es necesario reiniciar el servidor mediante el comando `systemctl restart ssh` para aplicar los cambios.

## Referencias

* [SSH Key Best Practices](https://dev.to/paulmicheli/ssh-key-best-practices-2cb7)
* [SSH Best Practices using Certificates, 2FA and Bastions](https://goteleport.com/blog/how-to-ssh-properly/)
