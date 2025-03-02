---
layout: post
blog-width: true
title: Obtener una línea de comandos como SYSTEM en Windows 7
date: 2011-03-03 15:21:35
published: true
tags:
- Windows
author:
  display_name: Manel Rodero
---

Hay veces en las que es necesario tener acceso al mismo entorno en el que se ejecuta la cuenta SYSTEM (por ejemplo para comprobar que un _script_ que va a ser programado bajo este usuario funciona correctamente).

En Windows 2000, 2003 y XP se podía usar el comando AT para obtener una línea de comandos en esta cuenta ejecutando `AT hh:mm /interactive cmd.exe`. ¿Por qué funcionaba así? Por una razón muy sencilla. El comando anterior lo único que ha hecho es programar la ejecución de una línea de comandos a una hora concreta. Como el servicio Windows que se encarga de la programación de tareas se ejecuta bajo SYSTEM esta tarea también lo hará. Al añadir el parámetro `/interactive` conseguimos ver el resultado por pantalla y tener acceso a la línea de comandos.

En Windows 2008, Vista y Windows 7, debido a la seguridad UAC y al hecho de tener la sesión de usuario y la de servicios en sesiones diferentes, este truco no funciona y hay que buscar alternativas. La más sencilla es usar la utilidad [PsExec][1] de SysInternals (Microsoft) y ejecutar el siguiente comando:

```plaintext
C:TEMP>psexec -i -d -s cmd.exe

PsExec v1.98 - Execute processes remotely  
Copyright (C) 2001-2010 Mark Russinovich  
Sysinternals - www.sysinternals.com

cmd.exe started on MACHINE with process ID 5428.
```

Se abrirá una nueva ventana con la línea de comandos ejecutándose desde la cuenta SYSTEM (se puede comprobar ejecutando el comando `whoami.exe`) desde la que podremos comprobar sin problemas los _scripts_ ;-)

[1]: https://learn.microsoft.com/en-us/sysinternals/downloads/psexec
