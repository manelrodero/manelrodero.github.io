---
layout: post
blog-width: true
title: '"Baúl de Notas" se muda a Dreamhost'
date: 2006-10-08 19:49:34.000000000 +02:00
last-updated: '2025-03-01 18:36:11'
published: true
tags:
- WordPress
author:
  display_name: Manel Rodero
---

Hoy por fin he podido ponerme y he acabado la migración de "Baúl de Notas" desde GoDaddy.com hacia [Dreamhost.com][1]{:target="_blank"}.

Ha sido un proceso sencillo gracias a las explicaciones de [Carlos Carmona][2]{:target="_blank"} y [Jordi Abad][3]{:target="_blank"} que ya habían pasado antes por este proceso.

También me han sido de utilidad los enlaces que me han enviado por correo electrónico el equipo de soporte de Dreamhost:

* [DNS - Viewing site before DNS change][4]{:target="_blank"}
* [DNS - Accessing your Database before DNS change][5]{:target="_blank"}

Al final, únicamente he tenido algunos problemas con los acentos (tanto en el blog como en sabrosus) por la conversión entre "latin1" y "utf8". Por lo que he visto _googleando_ suele ser el problema habitual al exportar/importar bases de datos MySQL, sobretodo si se trata de versiones distintas de la aplicación como en mi caso (4.0.27 en GoDaddy y 5.0.24 en Dreamhost).

Ahora, únicamente me queda esperar a ver qué tal funciona este proveedor. Al menos espero que funcione igual o mejor que GoDaddy con los que nunca tuve ningún problema, al contrario, estaba bastante contento con el servicio que me prestaban (bueno, y que me prestan, ya que el registro de los dominios sigue estando allí).

Los motivos fundamentales de este cambio han sido los siguientes:

* A final de este mes se acababa el año de _hosting_ que tenía con GoDaddy y tenía que renovar el plan de alojamiento.
* En GoDaddy, si quería alojar múltiples dominios en el mismo espacio de disco del _hosting_ tenía que pasarme al plan Deluxe (ahora tenía el plan Economy).
* En GoDaddy los múltiples dominios alojados en el plan no son completamente independientes. Hay un dominio principal y sus ficheros están en el directorio raíz del espacio de disco del usuario. Cada nuevo dominio se aloja en un subdirectorio de éste. El problema es que se puede acceder a los ficheros de un dominio secundario usando el nombre del dominio principal y el nombre del subdirectorio.

De momento, he notado una mejoría en el acceso a la web de administración de WordPress y al blog en general. Esperemos que no sean suposiciones mías ;-)

### Historial de cambios

* **2006-10-08**: Documento inicial
* **2025-02-28**: Corrección de enlaces (WebArchive.org)

[1]: http://www.dreamhost.com/
[2]: https://web.archive.org/web/20061007093555/http://www.dkt.es/
[3]: https://web.archive.org/web/20061119044131/http://www.unblogmas.com/
[4]: https://web.archive.org/web/20061025190550/http://wiki.dreamhost.com/index.php/DNS_-_Viewing_site_before_DNS_change
[5]: https://web.archive.org/web/20061025190804/http://wiki.dreamhost.com/index.php/DNS_-_Accessing_your_Database_before_DNS_change
