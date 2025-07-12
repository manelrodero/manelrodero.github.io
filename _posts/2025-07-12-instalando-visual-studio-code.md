---
layout : post
blog-width: true
title: 'Instalando Visual Studio Code'
date: '2025-07-12 13:59:21'
#last-updated: '2025-07-12 13:59:21'
published: true
tags:
- Software
author:
  display_name: Manel Rodero
#cover-img: "/assets/img/blog/2025-07-12_cover.png"
thumbnail-img: ""
---

Visual Studio Code (VS Code) es un **editor de código fuente gratuito, ligero y multiplataforma** desarrollado por Microsoft. Diseñado para ser ágil pero potente, permite escribir, depurar y gestionar código en una amplia variedad de lenguajes como JavaScript, Python, C++, Java, HTML, entre muchos otros.

# Instalación portable

Aunque [Visual Studio Code](https://code.visualstudio.com/){:target="_blank"} se puede instalar a nivel de **sistema** (`Program Files`) o a nivel de **usuario** (`AppData`), yo prefiero instalarlo en modo [**portable**](https://code.visualstudio.com/docs/editor/portable){:target="_blank"} para que la configuración y las extensiones se guarden junto al programa.

La instalación se realiza de la siguiente manera:

* Descargar el fichero [ZIP](https://code.visualstudio.com/docs/?dv=winzip){:target="_blank"}
* Descomprimirlo en un directorio local (p.ej. `D:\SOFT\VSCode`).
* Crear un subdirectorio `D:\SOFT\VSCode\data`

La existencia de este directorio `data` indica a Visual Studio Code que debe guardar ahí la **configuración** y las **extensiones** que se instalen.

**Nota**: El único inconveniente de este método tenemos que encargarnos nosotros mismos de las actualizaciones, descargando la nueva versión y descomprimiéndola en el mismo directorio.

| Elemento | Instalación usuario | Instalación portable |
| --- | --- | --- |
| Settings | `%APPDATA%\Code` | `.\data\user-data` |
| Extensions | `%USERPROFILE%\.vscode\extensions` | `.\data\extensions` |

# Componentes adicionales

Si se utilizará Visual Studio para desarrollo, es recomendable instalar también los siguientes componentes adicionales:

* [Windows Subsystem for Linux](https://learn.microsoft.com/en-us/windows/wsl/install){:target="_blank"} (WSL)
* [Windows Terminal](https://apps.microsoft.com/detail/9n0dx20hk701?hl=en-US&gl=ES){:target="_blank"}
* [Git](https://git-scm.com/downloads){:target="_blank"}
* [PowerShell](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.5){:target="_blank"}
* [Container](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-containers){:target="_blank"}

```powershell
winget install --id Microsoft.WSL --source msstore
winget install --id Microsoft.WindowsTerminal --source msstore
winget install --id Git.Git --source winget
winget install --id Microsoft.PowerShell --source winget
```

# Wizard inicial

Cuando se ejecuta VSCode por primera vez, aparece un _wizard_ para realizar la configuración inicial.

Yo suelo realizar un mínima configuración y marcar todos los pasos como realizados:

* Use AI features with Copilot for free
  * Setup up Copilot
  * Sign-in with a GitHub account
  * Select user to authorize Visual Studio Code: `username`
  * Always allow `vscode.dev` to open links of this type
* Choose your theme &rarr; **Dark High Contrast**
* Rich support for all your languages:
  * [Python](https://code.visualstudio.com/docs/languages/python){:target="_blank"}
  * [Markdown](https://code.visualstudio.com/docs/languages/markdown){:target="_blank"}
  * [JSON](https://code.visualstudio.com/docs/languages/json){:target="_blank"}
  * [PowerShell](https://code.visualstudio.com/docs/languages/powershell){:target="_blank"}
  * [HTML](https://code.visualstudio.com/docs/languages/html){:target="_blank"}
  * [CSS](https://code.visualstudio.com/docs/languages/css){:target="_blank"}
  * [PHP](https://code.visualstudio.com/docs/languages/php){:target="_blank"}
  * YAML
* Tune your settings
* Unlock productivity with the Command Palette
* Watch video tutorials
* Code with extensions
* Install Git
* Customize your shortcuts

# [Extensiones](https://code.visualstudio.com/docs/getstarted/extensions){:target="_blank"}

Visual Studio Code tiene un [marketplace](https://code.visualstudio.com/docs/configure/extensions/extension-marketplace){:target="_blank"} para descargar e instalar extensiones que amplian las capacidades del editor.

Mara mi uso particular, suelo instalar las siguientes extensiones:

* [Container Tools](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-containers){:target="_blank"}
* [GitHub Copilot](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot){:target="_blank"}
* [GitHub Copilot Chat](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot-chat){:target="_blank"}
* [HTML CSS Suport](https://marketplace.visualstudio.com/items?itemName=ecmel.vscode-html-css){:target="_blank"}
* [Markdown PDF](https://marketplace.visualstudio.com/items?itemName=yzane.markdown-pdf){:target="_blank"}
* [markdownlint](https://marketplace.visualstudio.com/items?itemName=DavidAnson.vscode-markdownlint){:target="_blank"}
* [PowerShell](https://marketplace.visualstudio.com/items?itemName=ms-vscode.PowerShell){:target="_blank"}
* [Rainbow CSV](https://marketplace.visualstudio.com/items?itemName=mechatroner.rainbow-csv){:target="_blank"}
* [REG](https://marketplace.visualstudio.com/items?itemName=ionutvmi.reg){:target="_blank"}
* [Remote-SSH: Editing Configuration Files](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh-edit){:target="_blank"}
* [Remote Development](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack){:target="_blank"}
  * [WSL](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl){:target="_blank"}
  * [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers){:target="_blank"}
  * [Remote-SSH](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh){:target="_blank"}
  * [Remote -Tunnels](https://marketplace.visualstudio.com/items?itemName=ms-vscode.remote-server){:target="_blank"}
* [Remote Explorer](https://marketplace.visualstudio.com/items?itemName=ms-vscode.remote-explorer){:target="_blank"}
* [shell-format](https://marketplace.visualstudio.com/items?itemName=foxundermoon.shell-format){:target="_blank"}
* [XML Tools](https://marketplace.visualstudio.com/items?itemName=DotJoshJohnson.xml){:target="_blank"}

# [Configuración](https://code.visualstudio.com/docs/configure/settings){:target="_blank"}

La configuración de VSCode se divide en **user settings** (que aplican de forma global a cualquier instancia de VSCode que se ejecute) y **workspace settings** (únicamente aplican cuando se abre un determinado proyecto o _workspace_).

Las preferencias de usuario se modifican usando la opción `Preferences: Open User Settings` desde la paleta de comandos y se guardan en el fichero `.\data\user-data\User\setttings.json`.

Las preferencias de un workspace se modifican usando la opción `Preferences: Open Workspace Settings` desde la paleta de comandos y se guardan en un directorio `.vscode` dentro del proyecto, lo cual es útil para compartir la configuración a través de Git.

**Nota**: Hay configuraciones de usuario que no están disponibles en un _workspace_ (por ejemplo las que tienen relación con las actualizaciones o la seguridad).

La configuración se puede [sincronizar](https://code.visualstudio.com/docs/configure/settings-sync){:target="_blank"} entre diferentes instancias de VSCode, ya sea en el mismo equipo o en equipos diferentes.

En mi caso únicamente sincronizo las configuraciones marcadas en **negrita**:

* **Settings**
* Keyboard Shortcuts
* Snippets
* Tasks
* MCP Servers
* UI State
* **Extensions**
* Profiles
* Prompts and Instructions

# PowerShell

La configuración inicial de PowerShell, `Get started with PowerShell`, consta de las siguientes secciones que suelo marcar rápidamente como completadas:

* Choose a version of PowerShell
* Create a PowerShell file
* Switch sessions
* Try ISE mode
* Open the PowerShell Extension Terminal
* Explore more resources
		
# [Telemetria](https://code.visualstudio.com/docs/configure/telemetry){:target="_blank"}

Para evitar el envío de información a Microsoft sobre el uso, errores, etc. de VSCode, se puede desactivar la telemetría:

```json
"telemetry.telemetryLevel": "off"
```

# [WSL](https://code.visualstudio.com/docs/remote/wsl){:target="_blank"}

La configuración inicial de WSL, `Get Started with WSL, consta de las siguientes secciones que suelo marcar rápidamente como completadas:

* Get Started with WSL
* Open a WSL Windows

## Arquitectura VSCode con WSL

La arquitectura de VS Code con WSL es tipo cliente-servidor:

* El **cliente** se ejecuta en Windows (interfaz, temas, snippets)
* El **servidor** se ejecuta en WSL (extensiones que analizan código, linting, depuración, etc.)

Por eso, extensiones como **markdownlint**, que analizan archivos, o **Copilot Chat**, que interactúan con el código, necesitan estar instaladas en WSL para funcionar correctamente en ese entorno.

**Nota**: la configuración de este entorno se guarda en el fichero `~/.vscode-server/data/Machine/settings.json` del servidor remoto.

# `settings.json`

```json
{
    // Configuración del entorno global

    "workbench.colorTheme": "Default High Contrast", // Tema oscuro de alto contraste
    "workbench.startupEditor": "none",
    "window.zoomLevel": 1,
    "telemetry.telemetryLevel": "off", // Deshabilitar la telemetría
    "extensions.autoUpdate": false, // Deshabilitar la actualización automática de extensiones

    // Configuración del editor

    "editor.minimap.enabled": false, // Deshabilitar el minimapa
    "editor.tabSize": 2, // Tamaño de tabulación de 2 espacios
    "editor.insertSpaces": true, // Insertar espacios en lugar de tabulaciones
    "editor.inlineSuggest.enabled": true, // Habilitar sugerencias en línea
    "files.autoSave": "off", // Deshabilitar el guardado automático de archivos

    // Configuración según el tipo de archivo o lenguaje

    "files.associations": {
        "docker-compose.yml": "yaml"
    },

    "[yaml]": {
        "editor.tabSize": 2,
        "editor.insertSpaces": true
    },

    "[bat]": {
        "files.encoding": "windows1252"
    },

    "powershell.codeFormatting.preset": "OTBS", // Formato de código OTBS (Kernigan & Ritchie)

    // Configuraciones para la edición remota (WSL, SSH, Docker)

    "remote.SSH.remotePlatform": {
        "docker-root": "linux",
        "pi4nas-pi": "linux",
        "unraid-root": "linux"
    }
}
```

### Historial de cambios

* **2025-07-12**: Documento inicial
