---
layout : post
blog-width: true
title: 'Instalación de Cryptomator en modo portable'
date: '2021-12-24 14:06:05'
published: true
tags:
- Software
author:
  display_name: Manel Rodero
---

[**Cryptomator**](https://cryptomator.org/) es una herramienta de código abierto que permite encriptar el contenido de los ficheros y sus nombres utilizando el protocolo AES y claves de 256 bits.

Esta herramienta se puede utilizar para encriptar carpetas que sincronizamos en Google Drive, OneDrive, etc. y evitar que su contenido pueda ser leído por alguien que tengo acceso a las mismas.

A continuación se explica cómo instalarlo en modo portable (utilizando el [_port_ no oficial](https://portapps.io/app/cryptomator-portable/) de [CrazyMax](https://crazymax.dev/)) y cómo actualizarlo cuando aparezcan versiones oficiales más recientes.

# Descarga e instalación de la versión portable

El primer paso consistirá en descargar la versión [portable](https://portapps.io/app/cryptomator-portable/) más reciente para Windows x64. A día de hoy, es la 1.6.1.8 del 27 de octubre de 2021.

* Descargar y ejecutar [cryptomator-portable-win64-1.6.1-8-setup.exe](https://portapps.io/download/cryptomator-portable-win64-1.6.1-8-setup.exe)
* Pulsar el botón `Next` para comenzar el asistente de instalación
* Seleccionar la opción _I understand this notice_ y pulsar el botón `Next`
* Instalar en `C:\portapps\cryptomator-portable` y pulsar el botón `Next`
* Pulsar el botón `Install`
* No marcar la opción para ejecutarlo y pulsar el botón `Finish`

## Configuración

La primera vez que se ejecuta el programa, se creará un fichero de configuración [cryptomator-portable.sample.yml](https://portapps.io/doc/configuration/) que se puede modificar para adaptarlo a nuestras necesidades.

En nuestro caso, se creará el fichero `cryptomator-portable.yml` con el siguiente contenido para deshabilitar la creación de logs de la plataforma Portapps:

```
common:
  disable_log: true
  args: []
  env: {}
  app_path: ""
app:
  cleanup: true
```

## Creación de un **vault**

A continuación se crea un **vault** (directorio encriptado) de la siguiente manera:

* Seleccionar la opción _Add Vault_
* Seleccionar la opción _Create New Vault_
* Introducir un nombre (p.ej. `Facturas`) y pulsar el botón `Next`
* Seleccionar la opción _Custom location_
* Seleccionar un directorio por debajo de la instalación (p.ej. `C:\portapps\cryptomator-portable\data\vault`) y pulsar el botón `Next`
* Introducir una contraseña y repetirla para confirmar que la conocemos
* Seleccionar la opción _Yes please, better than sorry_ para generar una clave de recuperación y pulsar el botón `Create Vault`
* Copiar la clave de recuperación

{: .box-note}
**Nota**: Es muy recomendable copiar y guardar a buen recaudo (por ejemplo en un gestor de contraseñas como Bitwarden o KeePassXC) esta clave de recuperación por si olvidamos la contraseña.

* Pulsar el botón `Unlock now`
* Introducir la contraseña para desbloquear el acceso

Si todo funciona correctamente, se mapeara este _vault_ (directorio cifrado) como una unidad usando el protocolo **WebDav** (p.ej. `F:\` &rarr; `\\127.0.0.1@42427\DavWWWRoot\EPUU2WhI2muE`).

En la siguiente captura se puede observar como el fichero de ejemplo `WELCOME.rtf` es visible accediendo a través de esta unidad pero está cifrado si se accede a través del sistema de ficheros propio de Windows:

![Vault vs Sistema de Ficheros][1]

## Vault portable

Para conseguir que el _vault_ que se acaba de crear sea realmente portable es necesario cambiar la ruta absoluta por una **ruta relativa** modificando el fichero `settings.json` del directorio `data`.

El fichero original contendrá algo similar a ésto:

```
{
  "directories": [
    {
      "id": "EPUU2WhI2muE",
      "path": "C:\\portapps\\cryptomator-portable\\data\\vault\\Facturas",
      "displayName": "Facturas",
      ...
    }
  ],
  ...
}
```

Después de cerrar completamente el programa, se edita este fichero y se modifica la variable `path` para convertirla en una **ruta relativa** de la siguiente manera:

```
{
  "directories": [
    {
      "id": "EPUU2WhI2muE",
      "path": "..\\data\\vault\\Facturas",
      "displayName": "Facturas",
      ...
    }
  ],
  ...
}
```

A continuación, si se vuelve a ejecutar el programa, todo debería funcionar normalmente tal como se puede observar en la siguiente captura:

![Vault portable en ..\data\vault][2]

# Actualización con la versión oficial

Aunque [CrazyMax](https://crazymax.dev/) suele portar la versión oficial bastante rápido, hay veces en que puede tardar en hacerlo. En ese caso es posible actualizar la instalación portable utilizando los ficheros de la instalación oficial.

Por ejemplo, a día de hoy, la versión oficial es la 1.6.5 del 16 de diciembre de 2021. El "truco" para actualizar la aplicación portable es el siguiente:

* Descargar la [versión oficial](https://cryptomator.org/downloads/win/thanks/) en **Windows Sandbox** (o una máquina virtual)
* Ejecutar el instalador, en este caso `Cryptomator-1.6.5-x64.exe`
* Pulsar el botón `Next` para comenzar el asistente de instalación
* Seleccionar la opción _I accept the terms in the License Agreement_ y pulsar el botón `Next`
* Instalar en `C:\Program Files\Cryptomator\` y pulsar el botón `Next`
* Seleccionar la opción _Create start menu shortcut(s)_ y pulsar el botón `Next`
* Pulsar el botón `Install`
* Pulsar el botón `Finish`

A continuación se sustituye el contenido del directorio `app` de la instalación portable por el contenido de la instalación oficial tal como se muestra en la siguiente captura:

![Actualización con versión oficial][3]

Si se vuelve a ejecutar el programa, todo debería continuar funcionando con normalidad con la nueva versión instalada, tal como se muestra en la siguiente captura:

![Cryptomator portable actualizado][4]

[1]: /assets/img/blog/2021-12-24_image_1.png "Vault vs Sistema de Ficheros"
[2]: /assets/img/blog/2021-12-24_image_2.png "Vault portable en ..\data\vault"
[3]: /assets/img/blog/2021-12-24_image_3.png "Actualización con versión oficial"
[4]: /assets/img/blog/2021-12-24_image_4.png "Cryptomator portable actualizado"