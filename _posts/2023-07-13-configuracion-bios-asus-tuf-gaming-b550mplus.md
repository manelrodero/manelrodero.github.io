---
layout : post
blog-width: true
title: 'Configuración BIOS Asus TUF GAMING B550M-PLUS'
date: '2023-07-13 22:25:40'
last-updated: '2024-09-27 10:05:27'
published: true
tags:
- Hardware
author:
  display_name: Manel Rodero
---

Mi ordenador [**PCLyM 2021**](/about/desktop#pclym-2021) tiene los siguientes componentes principales:

* Placa base **Asus TUF GAMING B550M-PLUS**
* Procesador **AMD Ryzen 5 5600X 3.7GHz**
* Memoria **Kingston FURY Beast DDR4 3200 MHz 16GB 2x8GB CL16**

A fecha de hoy se ha actualizado la BIOS a la versión [**3607**](https://www.asus.com/motherboards-components/motherboards/tuf-gaming/tuf-gaming-b550m-plus/helpdesk_bios/?model2Name=TUF-GAMING-B550M-PLUS) del 22 de marzo de 2024 (anteriormente he tenido las versiones 3404, 3002 y 2423).

Al actualizar la BIOS no se mantiene la configuración existente y hay que volver a configurarla de la siguiente manera:

* Poner en marcha el ordenador
* Pulsar `F2` o `SUPR` para entrar en la BIOS
* Pulsar `F7` para acceder al modo avanzado
* Exit > Load Optimized Defaults > Pulsar `OK`
* Exit > Save Changes & Reset > Pulsar `OK`
* Volver a entrar en la BIOS
* Ai Tweaker > Ai Overclock Tuner > Auto &rarr; **D.O.C.P.**
  * D.O.C.P. = D.O.C.P. DDR4-3200 16-18-18-36-1.3 5V
  * Memory Frequency = Auto &rarr; DDR4-3200MHz
  * DRAM Timing Control
    * DRAM CAS# Latency: Auto &rarr; 16
    * Trcdrd: Auto &rarr; 18
    * Trcdwr: Auto &rarr; 18
    * DRAM RAS# PRE Time: Auto &rarr; 18
    * DRAM RAS# ACT Time: Auto &rarr; 36
  * DRAM Voltage: Auto &rarr; 1,35
* Advanced > CPU Configuration > SVM Mode > Disabled &rarr; **Enabled** (Virtualización)
* Advanced > APM Configuration > Power On By PCI-E > Disabled &rarr; **Enabled** (Wake on LAN)
* Monitor > Q-Fan Configuration > CPU Fan Profile > Standard &rarr; **Silent**
* Monitor > Q-Fan Configuration > Chassis Fan(s) Configuration > Chassis Fan 1 Profile > Standard &rarr; **Silent**
* Monitor > Q-Fan Configuration > Chassis Fan(s) Configuration > Chassis Fan 2 Profile > Standard &rarr; **Silent**
* Boot > Boot Configuration > Fast boot > Enabled &rarr; **Disabled**
* Boot > Boot Configuration > Setup Mode > **Advanced Mode**
* Boot > Secure Boot > OS Type > Other OS &rarr; **Windows UEFI mode**
* Tool > Asus Armoury Crate > Enabled &rarr; **Disabled** (Servicio AsusUpdateCheck)
* Exit > Save Changes & Reset > Pulsar `OK`

# Referencias

* [How to set VT (Virtualization Technology) in BIOS](https://www.asus.com/us/support/FAQ/1045141/)

### Historial de cambios

* **2023-07-13**: Documento inicial
* **2024-09-27**: Revisión
