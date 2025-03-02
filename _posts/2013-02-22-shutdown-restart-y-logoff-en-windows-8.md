---
layout: post
blog-width: true
title: Shutdown, Restart y Logoff en Windows 8
date: 2013-02-22 19:14:45
published: true
tags:
- Windows
author:
  display_name: Manel Rodero
---

En Windows 8, el acceso a las opciones de Shutdown, Restart y Logoff están más "ocultas" que en versions anteriores de Windows. Por ejemplo, para hacer un Shutdown, es necesario descubrir el Charm Menu, hacer clic en la Settings charm, hacer clic en el botón Power y después seleccionar Shut Down en el menú desplegable.

Para realizar estas acciones más rápidamente, se pueden crear accesos directos en el escritorio (Desktop), en la barra de tareas (Taskbar) o en la pantalla de inicio (Start screen). Ésto se puede hacer "a mano", mediante scripts en diferentes lenguajes (p.ej. PowerShell) o usando diversas utilidades (p.ej. [NirCmd][2]{:target="_blank"}, [IconsExtract][3]{:target="_blank"}).

```plaintext
nircmd shortcut c:\Windows\system32\shutdown.exe ~$folder.desktop$ "Shutdown" "/s /t 0" c:\Windows\system32\shell32.dll 28

nircmd shortcut c:\Windows\system32\shutdown.exe ~$folder.desktop$ "Restart" "/r /t 0" c:\Windows\system32\shell32.dll 16739

nircmd shortcut c:\Windows\system32\logoff.exe ~$folder.desktop$ "Logoff" "" c:\Windows\system32\shell32.dll 45
```

Una vez creados estos accesos directos, se pueden anclar a la barra de tareas (Pin to Taskbar) o a la pantalla de inicio (Pin to Start) para que sean accesibles en todo momento.

[2]: https://www.nirsoft.net/utils/nircmd.html "NirCmd"
[3]: https://www.nirsoft.net/utils/iconsext.html "IconsExtract"
