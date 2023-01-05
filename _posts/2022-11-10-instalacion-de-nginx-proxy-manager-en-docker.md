---
layout : post
blog-width: true
title: 'Instalación de Nginx Proxy Manager en Docker'
date: '2022-11-10 22:09:28'
published: true
tags:
- Docker
author:
  display_name: Manel Rodero
---

[Nginx Proxy Manager](https://nginxproxymanager.com/) es una interfaz de administración de [_Nginx_](https://nginx.org/en/) que permite exponer servicios de una manera sencilla y segura usando certificados de [Let's Encrypt](https://letsencrypt.org/).

La [instalación en Docker](https://hub.docker.com/r/jc21/nginx-proxy-manager) se realiza usando la imagen `jc21/nginx-proxy-manager`.

La forma más sencilla de hacerlo es usando un fichero [`docker-compose.yml`](https://nginxproxymanager.com/setup/) con el siguiente contenido:

```yaml
version: '3'
services:
  nginxpm:
    image: jc21/nginx-proxy-manager:latest
    container_name: nginxpm
    environment:
      TZ: 'Europe/Madrid'
      # DB_SQLITE_FILE: "/data/database.sqlite"
      DISABLE_IPV6: 'true'
    volumes:
      - ~/volumes/nginxpm/data:/data
      - ~/volumes/nginxpm/letsencrypt:/etc/letsencrypt
    ports:
      - '80:80' # Public HTTP Port
      - '443:443' # Public HTTPS Port
      - '81:81' # Admin Web Port
    restart: unless-stopped
```

> **Nota**: En este ejemplo se utiliza [SQLite](https://www.sqlite.org/) como motor de base de datos. Si se quisiera utilizar [MySQL](https://www.mysql.com/) o [MariaDB](https://mariadb.org/), habría que seguir las [instrucciones](https://nginxproxymanager.com/setup/#using-mysql-mariadb-database) para añadir la información de conexión al contenedor de estas bases de datos.

Se puede usar `docker-compose up -d` o usar el contenido del fichero en Portainer.

# Configuración inicial

Una vez en marcha, se puede acceder a Nginx Proxy Manager a través del puerto `81` (en este ejemplo [http://192.168.1.180:81](http://192.168.1.180:81)) y comenzar la configuración.

El usuario inicial es `admin@example.com` y su contraseña es `changeme`.

Una vez iniciada la sesión, se editan los datos del usuario:

* Full Name: `Administrator`
* Nickname: `Admin`
* Email: `nginx@mydomain.com`

Y, a continuación, se cambia la contraseña por otra más segura:

* Current Password: `changeme`
* New Password: `*********`
* Confirm Password: `*********`

# Certificados SSL

Para pedir un certificado SSL a través de [Let's Encrypt](https://letsencrypt.org/) hay que acceder a `http://192.168.1.180:81/nginx/certificates` y seguir el asistente:

> **Nota**: Antes de comenzar, hay que crear un registro DNS de tipo A para el dominio.

* Pulsar el botón `Add SSL Certificate`
* Rellenar el formulario:
  * Domain names: `registro.mydomain.com`
  * Email Address for Let's Encrypt: `nginx@mydomain.com`
  * Marcar la opción `Use a DNS Challenge`
  * Escoger el proveedor de DNS: `Cloudflare`
  * Introducir las credenciales para realizar la operación (en el caso de Cloudflare un _token_ que asigna permisos):

```
# Cloudflare API token
dns_cloudflare_api_token = 8-3x-wz_B7cZXjMNE3Bz1bLMa4YsYQ-XESthjXe3
```

* Marcar la opción para aceptar [Términos de Servicio de Let's Encrypt](https://letsencrypt.org/repository/)
* Pulsar el botón `Save`

Si todo va bien, después de unos segundos, se habrá creado un certificado SSL con una duración de 3 meses para el dominio:

![Certificado SSL][1]

> **Nota**: Nginx Proxy Manager se encargará de renovar el certificado de forma automática.

# Proxy Host

Para crear un _proxy host_ hay que acceder a `http://192.168.1.180:81/nginx/proxy` y seguir el asistente.

> **Nota**: Antes de comenzar, hay que crear un registro DNS de tipo A para el dominio.

* Pulsar el botón `Add Proxy Host`
* Rellenar el formulario de la pestaña `Details`
  * Domain names: `registro.mydomain.com`
  * Scheme: `http`
  * Forward Hostname/IP: `192.168.1.180`
  * Forward Port: `8989` (en el caso de [Sonarr](instalacion-de-sonarr-en-docker))
  * Habilitar `Block Common Exploits`
  * (Opcional) Habilitar `Websockets`
* Seleccionar el certificado SSL creado anteriormente en la pestaña `SSL`
  * SSL Certificate: `registro.mydomain.com`
  * Habilitar `Force SSL`
  * (Opcional) Habilitar `HTTP/2 Support`
  * (Opcional) Habilitar [`HSTS`](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security)
* Pulsar el botón `Save`

Si todo va bien, se habrá creado un _proxy host_ que redirige las peticiones de `https://registro.mydomain.com` a `http://192.168.1.180:8989` de forma transparente:

![Proxy Host][2]

# Docker _network_

Al tener servicios ejecutándose en la el mismo _Docker host_ que Nginx Proxy Manager, es recomendable crear una [_docker network_](https://nginxproxymanager.com/advanced-config/#best-practice-use-a-docker-network) **específica**.

De esta manera no será necesario publicar los puertos de los servicios a todas las interfaces de red del _Docker host_.

Primero habría que crear la red usando el comando `docker network create` tal como se muestra a continuación:

```Bash
# Red Interna
docker network create nginxpm_network
```

A continuación se añade al fichero `docker-compose.yml` de Nginx Proxy Manager y del resto de servicios que se ejecuten en el _Docker host_ la siguiente información:

```yaml
networks:
  default:
    external: true
    name: nginxpm_network
```

De esta manera se podría crear un _proxy host_ usando los nombres de los **servicios** Docker sin tener que exponer los puertos al _Docker host_ fuera de esta red.

> **Nota**: Dado que el _service name_ se utiliza como _hostname_, éstos tienen que ser únicos al usar la misma red.

# Registro DNS en Cloudflare

Para crear un registro DNS de tipo A en [Cloudflare](dns-dinamico-gratuito-usando-cloudflare) para `hostname`, se accede a `Websites` &rarr; `mydomain.com` &rarr; `DNS` &rarr; `Records` &rarr; `(+) Add record` y se introduce la información necesaria:

* Domain: `mydomain.com`
* Type: `A`
* Name: `hostname`
* IPv4 address: `192.168.1.180`
* Proxy status: `DNS only`

# Actualizar

Si ya se había instalado Nginx Proxy Manager anteriormente, se puede actualizar de la siguiente manera:

```
docker stop nginxpm
docker rm nginxpm
docker rmi jc21/nginx-proxy-manager
docker-compose up -d
```

# Soporte

Algunos comandos para gestionar la configuración del contenedor:

```
# Acceder al shell mientras el contenedor está ejecutándose
docker exec -it nginxpm /bin/bash

# Monitorizar los logs del contenedor en tiempo real
docker logs -f nginxpm
```

# Referencias

* [Nginx Proxy Manager - How-To Installation and Configuration](https://www.youtube.com/watch?v=P3imFC7GSr0) @ YouTube "Christian Lempa"
* [Dominio propio y Cloudflare: La mejor alternativa a DuckDNS](https://www.youtube.com/watch?v=kKwIIR-jylQ) @ YouTube "Un loco y su tecnología"
* [Certificados gratuitos con Nginx Proxy Manager, Let's Encrypt y DuckDNS](https://www.youtube.com/watch?v=0H6nDtahr5Y) @ YouTube "Un loco y su tecnología"

[1]: /assets/img/blog/2022-11-10_image_1.png "Certificado SSL"
[2]: /assets/img/blog/2022-11-10_image_2.png "Proxy Host"
