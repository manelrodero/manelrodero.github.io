---
layout: post
blog-width: true
title: '"Seguridad" wireless en un WAG54G V2'
date: 2006-08-16 21:55:52
last-updated: '2025-03-01 18:36:11'
published: true
categories:
tags:
- Ordenadores
author:
  display_name: Manel Rodero
---

Desde hace varios meses tenía pendiente revisar los parámetros de la configuración wireless de mi **Linksys WAG54G V2** para aumentar su "seguridad".

Los parámetros que se han modificado (en un _firmware_ versión 1.01.27Beta) son los siguientes:

* Wireless Network Mode : G-Only
* Wireless Network Name (SSID) : `*******`
* Wireless SSID Broadcast : Disable
* Security Mode : WPA Pre-Shared Key
* WPA Shared Key : `*******`
* Restrict Access – Permit Only – MAC Address Access List : XX:XX:XX:XX:XX:XX
* Authentication Type : Shared Key

Con estos valores se aumenta la "seguridad" de la red wireless, aunque, como se explica en el artículo [The six dumbest ways to secure a wireless LAN][1]{:target="_blank"}, sea únicamente un poquito ;-)

Referencia: [Easy 802.11g Security][2]{:target="_blank"}

### Historial de cambios

* **2006-08-16**: Documento inicial
* **2025-02-28**: Corrección de enlaces (WebArchive.org)

[1]: https://web.archive.org/web/20060830175936/http://blogs.zdnet.com/Ou/?p=43
[2]: https://web.archive.org/web/20061012164427/http://www.windowsitpro.com/Article/ArticleID/48168/48168.html
