---
layout: post
blog-width: true
title: Conocer la IP de una VM de Hyper-V
date: 2019-03-01 18:56:58.000000000 +01:00
published: true
tags:
- Windows
- Hyper-V
- PowerShell
author:
  display_name: Manel Rodero
---

Es bastante habitual utilizar la conexión directa a un VM en Hyper-V utilizando "Virtual Machine Connection" tal como se muestra en la siguiente captura:

![Virtual Machine Connection][1]

Ahora bien, es mucho más habitual conectarse mediante **Remote Desktop** y, por tanto, es necesario conocer la dirección IP.

Si la VM está configurada para obtener su dirección IP mediante DHCP ¿Cómo se puede obtener la dirección que se le ha asignado? Pues, lo más sencillo es utilizar el cmdlet `Get-VM` de PowerShell.

Para obtener las máquinas virtuales existentes en Hyper-V únicamente hay que ejecutar el siguiente cmdlet:  

    Get-VM

Al consultar las propiedades del objeto `Microsoft.HyperV.PowerShell.VirtualMachine` obtenido mediante el cmdlet anterior, se observa que interesa expandir la lista de adaptadores de red "**NetworkAdapters**":
    
    Get-VM | Select -ExpandProperty NetworkAdapters

Y se obtienen todas las propiedades de un adaptador, incluyendo la dirección IP que se buscaba:

![Obtención de la dirección IP de una VM usando PowerShell][2]

[1]: /assets/img/blog/2019-03-01_image_1.png "Virtual Machine Connection"
[2]: /assets/img/blog/2019-03-01_image_2.png "Obtención de la dirección IP de una VM usando PowerShell"
