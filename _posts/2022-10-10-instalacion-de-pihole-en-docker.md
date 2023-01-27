---
layout : post
blog-width: true
title: 'Instalación de Pi-hole en Docker'
date: '2022-10-10 20:21:33'
published: true
tags:
- HomeLab
- Docker
author:
  display_name: Manel Rodero
---

[Pi-hole](https://pi-hole.net/) es un _sinkhole_ de DNS que protege los dispositivos de contenido no deseado, sin instalar ningún software del lado del cliente, usando las siguientes tecnologías:

 * [`dnsmasq`](https://www.thekelleys.org.uk/dnsmasq/doc.html), un servidor DNS/DHCP ligero
 * [`lighttpd`](https://www.lighttpd.net/), un servidor web diseñado y optimizado para alto rendimiento
 
La [instalación en Docker](https://hub.docker.com/r/pihole/pihole) se realiza usando la imagen `pihole/pihole`.

La forma más sencilla es usar un fichero `docker-compose.yml` con el siguiente contenido:

```yaml
version: "3"
services:
  pihole:
    image: pihole/pihole:latest
    container_name: pihole
    environment:
      - TZ=Europe/Madrid
      - FTLCONF_LOCAL_IPV4=192.168.1.80
    volumes:
      - /home/pi/volumes/pihole/etc:/etc/pihole
      - /home/pi/volumes/pihole/dnsmasq.d:/etc/dnsmasq.d
    ports:
      - 53:53/tcp
      - 53:53/udp
      - 8080:80/tcp # Porque habitualmente el 80 estará ocupado por un dashboard o un proxy
    restart: always
```

**Nota**: Si se utiliza Pi-hole como [servidor DCHP](https://docs.pi-hole.net/docker/DHCP/) hay que cambiar algunas cosas, como per ejemplo [añadir la _capability_ `NET_ADMIN`](https://github.com/pi-hole/docker-pi-hole#note-on-capabilities).

Se puede usar `docker-compose up -d` o usar el contenido del fichero en Portainer.

# Configuración

Una vez en marcha, se puede acceder a Pi-hole a través del puerto `8080` (en este ejemplo [http://192.168.1.180:8080](http://192.168.1.180:8080)) y comenzar la configuración.

La contraseña inicial se genera de forma aleatoria, por lo que es necesario mirar cuál es mediante el comando `docker logs pihole | grep random`.

Lo primero y más importante es cambiar esa contraseña por otra que hayamos generado nosotros:

* Desde el _host_ ejecutar: `docker exec -it pihole /bin/bash`
* Desde el contenedor de Pi-hole ejecutar: `pihole -a -p` e introducir la nueva contraseña

A continuación hay que configurar el servidor de DNS _upstream_ que se quire utilizar:

* Acceder al menú `Settings`
* Acceder a la pestaña `DNS`
* Marcar los dos servidores de `Quad9 (filtered, DNSSEC)` en **IPv4**
  * 9.9.9.9
  * 149.112.112.112

# Listas

Si en algún momento es necesario borras las _blocklist_ se puede hacer desde terminal `sqlite3 /etc/pihole/gravity.db "DELETE FROM adlist"` o accediendo a la opción `Adlists` del menú lateral.

Para [restaurar la lista por defecto](https://discourse.pi-hole.net/t/restoring-default-pi-hole-adlist-s/32323) únicamente hay agregar la siguiente URL:

* `https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts`

Cada vez que se añaden o borran listas hay que actualizar **Gravity** (`Tools` &rarr; `Update Gravity`) para que se genere la base de datos completa, sin errores ni duplicados.

# Actualizar

Si ya se había instalado Pi-hole anteriormente, se puede actualizar de la siguiente manera:

```
docker stop pihole
docker rm pihole
docker rmi pihole/pihole
docker-compose up -d
```

# Soporte

Algunos comandos para gestionar la configuración del contenedor:

```
# Acceder al shell mientras el contenedor está ejecutándose
docker exec -it pihole /bin/bash

# Monitorizar los logs del contenedor en tiempo real
docker logs -f pihole
```

# Referencias

* [How to Set up Pihole in a Docker Container](https://adamtheautomator.com/pi-hole-in-docker/)
* [Setting up Pi-hole as a recursive DNS server (Explained)](https://sidmulajkar.com/posts/setting-up-pihole-as-a-recursive-dns-server/)
* [Pihole: Add custom DNS mappings](https://thiagowfx.github.io/2022/01/pihole-add-custom-dns-mappings/)
* [Avoid The Hack: The Best Pi-Hole Blocklists](https://avoidthehack.com/best-pihole-blocklists)
* [7 Best PiHole Blocklists – Safely Block all Ads](https://ninja-ide.org/pihole-blocklists-2022/)
