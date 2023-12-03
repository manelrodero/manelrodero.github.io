---
layout : post
blog-width: true
title: 'Instalación de WSL durante el OSD'
date: '2023-12-02 13:57:48'
published: true
tags:
- Windows
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2023-12-02_cover.png"
thumbnail-img: ""
---

[**Windows Subsystem for Linux** (WSL)](https://learn.microsoft.com/en-us/windows/wsl/){:target=_blank} permite ejecutar un entorno GNU/Linux, incluida la mayoría de las herramientas, utilidades y aplicaciones de línea de comandos, directamente en Windows, sin modificaciones y sin la sobrecarga de una máquina virtual tradicional o una configuración de arranque dual.

# Instalación _inbox_

Hasta ahora había usado el mismo método que se utiliza en la [instalación manual de WSL](https://learn.microsoft.com/en-us/windows/wsl/install-manual){:target=_blank} para agregar las siguientes [**características opcionales**](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/windows-features){:target=_blank} a nuestra imagen `WIM` de Windows 10/11 usando el comando [`dism.exe` de forma _offline_](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/enable-or-disable-windows-features-using-dism){:target=_blank}:

```
# Microsoft-Windows-Subsystem-Linux (WSL 1)
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

# VirtualMachinePlatform (WSL 2)
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

## WSL Update

El método anterior requiere que se instale la versión más reciente del [**WSL 2 Linux Kernel**](https://github.com/microsoft/WSL2-Linux-Kernel){:target=_blank} para poder ejecutar WSL dentro de la imagen del sistema operativo Windows.

El enlace a [`wsl_update_x64.msi`](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi){:target=_blank} para instalar el paquete `Windows Subsystem for Linux Update` debería apuntar siempre a la versión más reciente pero, a dia de hoy, únicamente instala la versión **5.10.16** tal como se muestra en la siguiente captura:

![WSL Update 5.10.16][8]

El último _kernel_ disponible a día de hoy a través de Windows Update es la versión **5.10.102.2** del 25 de marzo de 2022. Se puede descargar un fichero `*.cab` desde el [Microsoft Update Catalog](https://www.catalog.update.microsoft.com/Search.aspx?q=wsl){:target=_blank} para extraer el fichero `wsl_update_x64.msi` e instalarlo de forma desatendida.

En GitHub está disponible el código fuente de la versión estable más reciente del _kernel_ a día de hoy, la [**5.15.133.1**](https://github.com/microsoft/WSL2-Linux-Kernel/releases/tag/linux-msft-wsl-5.15.133.1){:target=_blank} del 7 de octubre de 2023.

Para compilar este código fuente usando las opciones de configuración optimizadas por Microsoft para que funcione correctamente en Windows/WSL 2 hay que hacer lo siguiente desde una distribución reciente de Ubuntu:

```
sudo install build-essential flex bison libssl-dev libelf-dev
make KCONFIG_CONFIG=Microsoft/config-wsl
```

## WSLg Preview

Además, sería interesante instalar el componente **WSLg** o [Windows Subsystem for Linux GUI](https://github.com/microsoft/wslg){:target=_blank} que permite habilitar el soporte para ejecutar aplicaciones Linux gráficas (X11 y Wayland) integradas totalmente en la experiencia de escritorio de Windows.

![glxgears en WSLg][10]

El enlace a [`wsl_graphics_support_x64.msi`](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_graphics_support_x64.msi){:target=_blank} para instalar el paquete `Windows Subsystem for Linux WSLg Preview` debería apuntar siempre a la versión más reciente pero, a dia de hoy, únicamente instala la versión **1.0.26** tal como se muestra en la siguiente captura:

![WSLg Preview 1.0.26][9]

En GitHub está disponible el código fuente de la versión estable más reciente de WSLg a día de hoy, la [**1.0.51**](https://github.com/microsoft/wslg/releases/download/v1.0.51/Microsoft.WSLg.1.0.51.zip){:target=_blank} del 24 de marzo de 2023.

# [Microsoft Store](https://aka.ms/wslstorepage){:target=_blank}

El 22 de noviembre de 2022 se anunció que el [Windows Subsystem for Linux estaría en la Microsoft Store](https://devblogs.microsoft.com/commandline/the-windows-subsystem-for-linux-in-the-microsoft-store-is-now-generally-available-on-windows-10-and-11/){:target=_blank} para facilitar su actualización sin tener que esperar a una actualización del sistema operativo Windows.

![WSL en Microsoft Store][11]

## WSL 2

Esto hace que los comandos `wsl.exe --install` y `wsl.exe --update` instalen WSL desde la Microsoft Store si no se indica lo contrario mediante el parámetro `--web-download`.

Este cambio ha causado muchas confusiones a la hora de "tener la versión más reciente de WSL" ya que puede haber problemas si se mezclan componentes instalados a nivel del sistema operativo con componentes instalados desde la Microsoft Store.

De hecho, al usar el comando `--install` para instalar WSL desde la Microsoft Store:

* no se habilita el componente opcional `Windows Subsystem for Linux`
* no se instalan los paquetes MSI `WSL kernel` y `WSLg` al no ser ya necesarios (están incluídos en el WSL _package_)
* se habilita el componente opcional `Virtual Machine Platform`
* se habilita el parámetro `--version` para mostrar información sobre la instalación

![Instalación WSL desde Microsoft Store][1]

{: .box-note}
**Nota**: El parámetro `--version` no existe si se está ejecutando WSL instalado como componente del sistema operativo, por lo que es una manera sencilla de saber el tipo de instalación que tenemos.

Después de reiniciar el equipo para completar la instalación de WSL se pueden usar los parámetros `--status` y `--version` para comprobar el estado y versiones tal como se muestra a continuación:

![WSL 2 Status/Version][3]

También se puede comprobar que únicamente se ha instalado la característica opcional `VirtualMachinePlatorm`:

![WSL 2 Features][4]

## WSL 1

En general interesa utilizar [WSL 2 en lugar de WSL 1](https://learn.microsoft.com/en-us/windows/wsl/compare-versions){:target=_blank} ya que aumenta el rendimiento del sistema de archivos y agrega compatibilidad total con las llamadas al sistema gracias al uso de un _kernel_ de Linux real.

Cuando es necesario [seguir usando WSL 1 por compatibilidad](https://learn.microsoft.com/en-us/windows/wsl/compare-versions#exceptions-for-using-wsl-1-rather-than-wsl-2){:target=_blank} se tiene que utilizar el parámetro `--enable-wsl1` para habilitarlo.

Algunos de los motivos para usar WSL 1 en lugar de WSL 2 son:

* Guardar ficheros del proyecto en Windows en lugar de Linux
* Acceder al puerto serie o USB
* Requerimientos de memoria estrictos
* Direccionamiento IP en la misma red (_bridge_) que el _host_

![Habilitar WSL 1 al instalar desde WSL desde la Microsoft Store][2]

Después de reiniciar el equipo para completar la instalación de WSL se pueden usar los parámetros `--status` y `--version` para comprobar el estado y versiones tal como se muestra a continuación:

![WSL 1 Status/Version][5]

{: .box-note}
**Nota**: Al habilitar WSL 1, también se instala el componente opcional `Windows Subsystem for Linux` que no se instala al relizar una instalación por defecto desde la Microsoft Store.

![WSL 1 Features][6]

# Problemas

Mientras realizaba diferentes pruebas para escribir este artículo he encontrado algunos problemas que pueden dificultar la instalación desatendida durante el OSD para un entorno multiusuario como el nuestro, donde los usuarios no tienen permisos de Administrador.

## UAC al instalar WSL

A día de hoy, aún teniendo permisos de Administrador, aparece un mensaje de UAC para el "Host Process for Windows Services" durante la instalación de WSL desde la Microsoft Store tal como muestro en el siguiente vídeo:

<iframe width="640" height="360" src="https://www.youtube-nocookie.com/embed/QgdMSswiS5o" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen="1"></iframe>

{: .box-note}
**Nota**: En el repositorio de WSL en GitHub hay abierta la [_issue 9032_](https://github.com/microsoft/WSL/issues/9032){:target=_blank} sobre este problema.

## Cada usuario necesita su _kernel_

La instalación inicial de WSL desde la Microsoft Store realizada por un usuario con permisos de Administrador no sirve para el resto de usuarios del ordenador.

Cada usuario necesita instalar los diferentes componentes de WSL, como el _kernel_, tal como se indica en los mensajes mostrados en la siguiente captura de pantalla:

![Cada usuario necesita su Kernel de WSL 2][7]

Si un usuario sin permisos de Administrador ejecuta `wsl.exe --update`, aparecerá un mensaje de UAC para introducir las credenciales de un usuario que sí los tenga. Si el usuario cancela esta elevación de permisos, la instalación fallará al 90% con el código de error `0x80240066`.

Si el usuario intenta instalar una distribución ejecutando `wsl.exe --install -d Ubuntu`, ésta parece que se instala correctamente pero falla con el código de error `0x800701bc`. Esto significa que WSL 2 requiere la actualización del componente _kernel_ que no se ha podido realizar anteriormente por falta de permisos.

## Instalar WSL requiere elevación

La ejecución del comando `wsl.exe --install` desde un usuario sin permisos de Administrador en un ordenador sin WSL (es decir, sin las características opcionales ni el _kernel_ instaladas previamente) hace aparecer un mensaje de UAC para elevar los permisos.

## Diferentes ubicaciones del _kernel_

El _kernel_ instalado usando el `MSI` o las actualizaciones directas desde Windows Update se encuentra en `C:\Windows\System32\lxss\tools\kernel`.

En cambio, si se ha instalado a través de la Microsoft Store o la descarga web desde GitHub, se encuentra en `C:\Program Files\WSL\tools\kernel`.

{: .box-note}
**Nota**: Se puede utilizar el fichero [`.wslconfig`](https://learn.microsoft.com/en-us/windows/wsl/wsl-config){:target=_blank} para indicar cuál de ellos se quiere utilizar.

# Soluciones

## Evitar instalar desde la Microsoft Store

Para evitar la instalación de componentes desde la Microsoft Store es necesario:

* Instalar las características opcionales usando `dism.exe`
* Instalar el _kernel_ de WSL2 utilizando el fichero `MSI` (descargado desde el catálogo de WU)
* Instalar WSLg utilizando el fichero `MSI`
* Activar la opción `Receive updates for other Microsoft products`

Con esta instalación, un usuario sin permisos de Administrador puede **instalar** una distribución usando el comando `wsl.exe --install -d <Distro>` y que funcionen las aplicaciones gráficas.

Y, además, puede **actualizar** WSL desde la Microsof Store, usando el comando `wsl.exe --update --web-download` (aunque la ayuda indique lo contrario tal como [he explicado en este hilo de X](https://twitter.com/manelrodero/status/1731325541981020454){:target=_blank}):

![WSL Inbox to Store][12]

## Usar MakeMeAdmin

Otra opción, para evitar trabajar siempre con permisos de Administrador, es usar [MakeMeAdmin](https://github.com/pseymour/MakeMeAdmin){:target=_blank}, una aplicación de código abierto que permite elevar temporalmente los permisos de una cuenta de usuario estándard al nivel del Administrador.

La instalación de cualquier componente de WSL que requiera elevación, se haría mediante la ayuda de MakeMeAdmin para tener los permisos necesarios.

# Referencias

* [Windows Subsystem for Linux Documentation](https://learn.microsoft.com/en-us/windows/wsl/){:target=_blank}
* [Troubleshooting Windows Subsystem for Linux](https://learn.microsoft.com/en-us/windows/wsl/troubleshooting){:target=_blank}
* [Craig Loewen, @craigaloewen](https://twitter.com/craigaloewen){:target=_blank}, Product Manager at Microsoft

[1]: /assets/img/blog/2023-12-02_image_1.png "Instalación WSL 2 desde Microsoft Store"
[2]: /assets/img/blog/2023-12-02_image_2.png "Habilitar WSL 1 al instalar desde WSL 2 desde la Microsoft Store"
[3]: /assets/img/blog/2023-12-02_image_3.png "WSL 2 Status/Version"
[4]: /assets/img/blog/2023-12-02_image_4.png "WSL 2 Features"
[5]: /assets/img/blog/2023-12-02_image_5.png "WSL 1 Status/Version"
[6]: /assets/img/blog/2023-12-02_image_6.png "WSL 1 Features"
[7]: /assets/img/blog/2023-12-02_image_7.png "Cada usuario necesita su Kernel de WSL 2"
[8]: /assets/img/blog/2023-12-02_image_8.png "WSL Update 5.10.16"
[9]: /assets/img/blog/2023-12-02_image_9.png "WSLg Preview 1.0.26"
[10]: /assets/img/blog/2023-12-02_image_10.png "glxgears en WSLg"
[11]: /assets/img/blog/2023-12-02_image_11.png "WSL en Microsoft Store"
[12]: /assets/img/blog/2023-12-02_image_12.png "WSL Inbox to Store"