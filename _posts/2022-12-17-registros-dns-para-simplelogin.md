---
layout : post
blog-width: true
title: 'Registros DNS para SimpleLogin'
date: '2022-12-17 07:28:23'
published: true
tags:
- Seguridad
author:
  display_name: Manel Rodero
---

A la hora de utilizar un dominio propio con [SimpleLogin](https://simplelogin.io/) será necesario registrar diferentes registros en nuestro DNS tal como se muestra en la siguiente figura:

![Registros DNS][1]

# Registro MX

Son los 2 registros necesarios para que SimpleLogin pueda gestionar el correo electrónico de nuestro dominio `midominio.com`:

* Destino: `mx1.simplelogin.co.`
  * Tipo = MX
  * Prioridad = 10
  * Dominio = `midominio.com` o `@`
* Destino: `mx1.simplelogin.co.`
  * Tipo = MX
  * Prioridad = 20
  * Dominio = `midominio.com` o `@`

> Nota: Puede ser necesario eliminar el `.` en algunos registradores de dominios. El _root domain_ en algunos servicios se denota mediante `@` en lugar de utilizar `midominio.com`.

# SPF (Registro TXT)

Se recomienda activar [SPF](https://en.wikipedia.org/wiki/Sender_Policy_Framework) para detectar la falsificación de direcciones de remitentes durante la entrega del correo electrónico.

De esta manera, se reduce la posibilidad de que nuestros correos electrónicos terminen en la carpeta de correo no deseado del destinatario.

* Valor: `v=spf1 include:simplelogin.co ~all`
  * Tipo = TXT
  * Dominio = `midominio.com` o `@`

# DKIM (Registro CNAME)

Se recomienda activar [DKIM](https://en.wikipedia.org/wiki/DomainKeys_Identified_Mail), un método de autenticación de correo electrónico diseñado para evitar la suplantación de correo electrónico.

De esta manera, se reduce la posibilidad de que nuestros correos electrónicos terminen en la carpeta de correo no deseado del destinatario.

* Valor: `dkim._domainkey.simplelogin.co.`
  * Tipo = CNAME
  * Dominio = `dkim._domainkey`

* Valor: `dkim02._domainkey.simplelogin.co.`
  * Tipo = CNAME
  * Dominio = `dkim02._domainkey`

* Valor: `dkim03._domainkey.simplelogin.co.`
  * Tipo = CNAME
  * Dominio = `dkim03._domainkey`

> Nota: Algunos registradores de DNS pueden requerir una ruta de registro completa, `dkim._domainkey.midominio.com` como valor de dominio. Si se utiliza Cloudflare **no hay que usar** la opción de _proxy_.

# DMARC (Registro TXT)

Se recomienda activar [DMARC](https://en.wikipedia.org/wiki/DMARC) para proteger el dominio del uso no autorizado, comúnmente conocido como suplantación de identidad por correo electrónico.

Construida alrededor de SPF y DKIM, una política DMARC le dice al servidor de correo receptor qué hacer si no se pasa ninguno de esos métodos de autenticación.

* Valor: `v=DMARC1; p=quarantine; pct=100; adkim=s; aspf=s`
  * Tipo = TXT
  * Dominio = `_dmarc`

> Nota: Algunos registradores de DNS pueden requerir una ruta de registro completa, `_dmarc.midominio.com` como valor de dominio.

[1]: /assets/img/blog/2022-12-17_image_1.png "Registros DNS"
