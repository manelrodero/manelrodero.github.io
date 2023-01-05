---
layout: post
blog-width: true
title: Descubrir PowerShell poco a poco
date: 2019-02-16 21:27:30.000000000 +01:00
published: true
tags:
- PowerShell
author:
  display_name: Manel Rodero
---

Lo mejor para aprender PowerShell es practicar cada día, pero también hay un truco que permite descubrir **cmdlets** nuevos cada día.

La idea es añadir al fichero de perfil **$profile** los siguientes comandos para que se ejecuten cada vez que se abra una consola de PowerShell:
    
    
    Get-Command -Module Microsoft*,Cim*,PS*,ISE | Get-Random | Get-Help -ShowWindow
    Get-Random -input (Get-Help about*) | Get-Help -ShowWindow
    

Para añadirlos al fichero de perfil, primero se comprueba si ya se tiene uno:
    
    
    Test-Path $profile
    

Si el resultado es **False**, entonces se crea un nuevo fichero de perfil mediante el siguiente comando:
    
    
    New-Item -Path $profile -Type File –Force
    

![Creación del fichero de perfil de PowerShell][1]

A continuación se edita el fichero (con Notepad o similar) y se añaden las líneas indicadas anteriormente. Al abrir una consola de PowerShell aparecerán un par de ventanas con contenido aleatorio sobre Powershell: un **cmdlet** perteneciente a los módulos más importantes de PowerShell y un **about_*** con los conceptos del lenguaje:

![Conceptos aleatorios sobre PowerShell][2]  

**Referencia**: [Jeff Hicks][3] nos proporciona más recursos básicos para aprender PowerShell en su post "[Essential PowerShell Resources][4]".

[1]: /assets/img/blog/2019-02-16_image_1.png "Creación del fichero de perfil de PowerShell"
[2]: /assets/img/blog/2019-02-16_image_2.png "Conceptos aleatorios sobre PowerShell"
[3]: https://twitter.com/JeffHicks
[4]: http://jdhitsolutions.com/blog/essential-powershell-resources/
