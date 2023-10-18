---
layout : post
blog-width: true
title: 'Configuración BIOS Asus TUF GAMING B550M-PLUS'
date: '2023-07-13 22:25:40'
published: true
tags:
- Hardware
author:
  display_name: Manel Rodero
---

Mi ordenador [**PCLyM 2021**](/about/desktop#pclym-2021) tiene los siguientes comonentes principales:

* Placa base **Asus TUF GAMING B550M-PLUS**
* Procesador **AMD Ryzen 5 5600X 3.7GHz**
* Memoria **Kingston FURY Beast DDR4 3200 MHz 16GB 2x8GB CL16**

A fecha de hoy se ha actualizado la BIOS a la versión [**3002**](https://www.asus.com/motherboards-components/motherboards/tuf-gaming/tuf-gaming-b550m-plus/helpdesk_bios/?model2Name=TUF-GAMING-B550M-PLUS) del 23 de febrero de 2023 (anteriormente tenía la 2423).

La BIOS se ha configurado de la siguiente manera:

* Poner en marcha el ordenador
* Pulsar `F2` o `SUPR` para entrar en la BIOS
* Pulsar `F7` para acceder al mozo avanzado
* Exit > Load Optimized Defaults > Pulsar `OK`
* Exit > Save Changes & Exit > Pulsar `OK`
* Volver a entrar en la BIOS
* Ai Tweaker > Ai Overclock Tuner > Auto &rarr; **D.O.C.P.**
  * D.O.C.P. = D.O.C.P. DDR4-3200 16-18-18-36-1.3 5V
  * Memory Frequency = Auto &rarr; D.O.C.P.
  * DRAM CAS#: Auto &rarr; 16
  * Trcdrd: Auto &rarr; 18
  * Trcdwr: Auto &rarr; 18
  * DRAM RAS# PRE Time: Auto &rarr; 18
  * DRAM RAS# ACT Time: Auto &rarr; 36
  * DRAM Voltage: Auto &rarr; 1,35
* Advanced > CPU Configuration > SVM Mode > Disabled &rarr; **Enabled**
* Advanced > APM Configuration > Power On By PCI-E > Disabled &rarr; **Enabled**
* Monitor > CPU Fan Profile > Standard &rarr; **Silent**
* Monitor > Chassis Fan(s) Configuration > Chassis Fan 1 Profile > Standard &rarr; **Silent**
* Monitor > Chassis Fan(s) Configuration > Chassis Fan 2 Profile > Standard &rarr; **Silent**
* Boot > Boot Configuration > Fast boot > Enabled &rarr; **Disabled**
* Boot > Boot Configuration > Setup Mode > **Advanced Mode**
* Boot > Secure Boot > OS Type > Other OS &rarr; **Windows UEFI mode**
* Exit > Save Changes & Exit > Pulsar `OK`

# Referencias

* [How to set VT (Virtualization Technology) in BIOS](https://www.asus.com/us/support/FAQ/1045141/)
