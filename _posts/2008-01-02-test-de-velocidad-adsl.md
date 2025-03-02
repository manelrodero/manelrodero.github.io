---
layout: post
blog-width: true
title: Test de velocidad ADSL
date: 2008-01-02 23:02:15
published: true
tags:
- Hardware
author:
  display_name: Manel Rodero
---

El resultado del test de velocidad ADSL, después de haber "optimizado" los parámetros TCP/IP de mi Windows XP SP2 recién instalado, es el siguiente:

![10Mbps][1]

Algunos de los parámetros TCP/IP que se han cambiado o modificado en el registro de Windows (clave `HKLM\SYSTEM\CurrentControlSet`) son los siguientes:

**Valores añadidos:**

```plaintext
ServicesDnscacheParametersMaxNegativeCacheTtl: 0x00000000  
ServicesDnscacheParametersNetFailureCacheTime: 0x00000000  
ServicesDnscacheParametersNegativeSOACacheTime: 0x00000000  
ServiceslanmanserverparametersSizReqBuf: 0x00004000  
ServicesTcpipParametersTcpWindowSize: 0x0007D780  
ServicesTcpipParametersGlobalMaxTcpWindowSize: 0x0007D780  
ServicesTcpipParametersEnablePMTUDiscovery: 0x00000001  
ServicesTcpipParametersEnablePMTUBHDetect: 0x00000000  
ServicesTcpipParametersSackOpts: 0x00000001  
ServicesTcpipParametersDefaultTTL: 0x00000040  
ServicesTcpipParametersTcpMaxDupAcks: 0x00000002  
ServicesTcpipParametersTcp1323Opts: 0x00000001  
ServicesTcpipParametersInterfaces{IDentificador}MTU: 0x000005DC
```

**Valores modificados:**

```plaintext
ServicesTcpipServiceProviderDnsPriority: 0x000007D0  
ServicesTcpipServiceProviderDnsPriority: 0x00000007  
ServicesTcpipServiceProviderHostsPriority: 0x000001F4  
ServicesTcpipServiceProviderHostsPriority: 0x00000006  
ServicesTcpipServiceProviderLocalPriority: 0x000001F3  
ServicesTcpipServiceProviderLocalPriority: 0x00000005  
ServicesTcpipServiceProviderNetbtPriority: 0x000007D1  
ServicesTcpipServiceProviderNetbtPriority: 0x00000008
```

Para modificar estos valores se ha usado el programa "[TCP Optimizer 2.0.3][2]{:target="_blank"}" y para realizar el test de velocidad se ha usado [SpeedTest.net][3]{:target="_blank"}, seleccionando el servidor más próximo a Castelldefels en estos momentos ([Barcelona, XTEC][4]{:target="_blank"}).

La hoja de _status_ del router ADSL que tengo actualmente (un D-Link DSL-G624T proporcionado por Ya.com con su paquete "hasta 20 megas con llamadas incluídas") indica los siguientes valores, constantes desde la contratación del servicio en Marzo de 2007:

**ADSL status shows the ADSL physical layer status:**

```plaintext
ADSL Firmware Version: 6.02.00.05 – 6.02.00.04 – 6.02.00.113 Annex A – 01.07.2c – 0.54  
ADSL Software Version: V3.00B01T01.YA-C.20061122  
Line State Connected  
Modulation ADSL_2plus  
Annex Mode Annex A  
Max Tx Power -38 dBm/Hz

SNR Margin (Downstream 12 dB, Upstream 12 dB)  
Line Attenuation (Downstream 12 dB, Upstream 5 dB)  
Data Rate (Downstream 11898 kbps, Upstream 944 kbps)
```

La prueba "definitiva" consistente en descargar un fichero mediante HTTP desde un servidor rápido (por ejemplo la actualización de Nero 8.2.8.0) usando un gestor de descargas (en este caso DownThemAll! 0.9.9.10 para Mozilla Firefox 2.0.0.11).

Tal como se ve en la siguiente imagen se alcanza el valor teórico del 80-90% (el 10-20% restante se "gasta" a causa del _overhead_ intrínseco del protocolo TCP/IP) de la velocidad de descarga a la que ha sincronizado mi router según la hoja de _status_ mostrada anteriormente:

![Descarga_Nero8_a_10Mbps_DuMeter][6]

[1]: /assets/img/blog/2008-01-02_image_1.png
[2]: http://www.speedguide.net/tcpoptimizer.php
[3]: http://www.speedtest.net/
[4]: http://www.xtec.cat/
[6]: /assets/img/blog/2008-01-02_image_2.jpg
