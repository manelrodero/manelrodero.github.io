---
layout: post
blog-width: true
title: Añadir RSAT en Windows 10 1809
date: 2019-02-20 19:12:25.000000000 +01:00
published: true
tags:
- Software
author:
  display_name: Manel Rodero
---

Las herramientas de administración remota del servidor para **Windows 10 1803 o anteriores** [se tienen que descargar desde Microsoft][1].

A partir de **Windows 10 1809** se han añadido como una característica bajo demanda (_Feature on Demand_) por lo que se puede instalar directamente desde **Settings > Apps > Manage optional features > Add a feature** tal como se muestra en la siguiente pantalla:

![RSAT como Feature on Demand en Windows 10 1809][2]

Para comprobar si están instaladas en un equipo, se puede utilizar el cmdlet **Get-WindowsCapability**:
    
    
    Get-WindowsCapability -Name RSAT* -Online | Select-Object -Property DisplayName, State
    

![Listado de capacidades RSAT][3]

En el ejemplo anterior, se puede observar que no hay ningún componente RSAT instalado en el equipo (su estado es **NotPresent**).

Para instalar un componente de RSAT, se puede utilizar el cmdlet **Add-WindowsCapability** utilizando el nombre del componente a instalar:
    
    
    Add-WindowsCapability –Online –Name "Rsat.Dns.Tools~~~~0.0.1.0"
    

Para instalar todos los componentes de una vez, se pueden concatenar los _cmdlets_ en la _pipeline_ de PowerShell:
    
    
    Get-WindowsCapability -Name RSAT* -Online | Add-WindowsCapability –Online
    

Si el equipo está configurado para utilizar un servidor WSUS en lugar de Windows Update, lo más probable es que aparezca el error "_Add-WindowsCapability failed. Error code = 0x800f0954_". En ese caso hay que **desactivar temporalmente el uso de WSUS** mediante el siguiente código:
    
    $currentWU = Get-ItemProperty -Path "HKLM:SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" -Name "UseWUServer" | Select -ExpandProperty UseWUServer
    Set-ItemProperty -Path "HKLM:SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" -Name "UseWUServer" -Value 0
    Restart-Service wuauserv
    Get-WindowsCapability -Name RSAT* -Online | Add-WindowsCapability –Online
    Set-ItemProperty -Path "HKLM:SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" -Name "UseWUServer" -Value $currentWU
    Restart-Service wuauserv

[1]: https://www.microsoft.com/en-us/download/details.aspx?id=45520
[2]: /assets/img/blog/2019-02-20_image_1.png "RSAT como Feature on Demand en Windows 10 1809"
[3]: /assets/img/blog/2019-02-20_image_2.png "Listado de capacidades RSAT"
