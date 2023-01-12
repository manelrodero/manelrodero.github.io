---
layout : post
blog-width: true
title: 'Instalación de Vaultwarden en Docker'
date: '2022-11-13 18:20:26'
published: true
tags:
- HomeLab
- Docker
author:
  display_name: Manel Rodero
---

[Vaultwarden](https://github.com/dani-garcia/vaultwarden) es una implementación alternativa de la [API de Bitwarden](https://bitwarden.com/help/public-api/), un gestor de contraseñas de código abierto.

Al estar escrita en Rust, necesita mucho menos recursos que la instalación de [Bitwarden en Docker](https://bitwarden.com/help/install-on-premise-linux/) por lo que puede funcionar incluso en una Raspberry Pi 4.

La instalación de [Vaultwarden en Docker](https://hub.docker.com/r/vaultwarden/server) se realiza usando la imagen `vaultwarden/server`.

La forma más sencilla de hacerlo es usando un fichero `docker-compose.yml` con el siguiente contenido:

```yaml
version: '3'
services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    environment:
      TZ: 'Europe/Madrid'
      DOMAIN: 'https://vault.mydomain.com/'
    volumes:
      - ~/volumes/vaultwarden:/data
    restart: unless-stopped
networks:
  default:
    external: true
    name: nginxpm_network
```

> **Nota**: Nótese que no se expone ningún puerto al _Docker host_ y que se utiliza la misma red que [Nginx Proxy Manager](instalacion-de-nginx-proxy-manager-en-docker).

Para iniciar el contenedor, se puede usar `docker-compose up -d` o usar el contenido del fichero anterior en **Portainer**.

# Pre-requisitos

Antes de poder acceder a Vaultwarden para configurarlo es necesario:

* Crear un [registro DNS de tipo A en Cloudflare](instalacion-de-nginx-proxy-manager-en-docker#registro-dns-en-cloudflare): `vault.mydomain.com` &rarr; 192.168.1.180

![DNS Type A][1]

* Crear un [certificado SSL de Let's Encrypt](instalacion-de-nginx-proxy-manager-en-docker#certificado-ssl) para el _hostname_ anterior

![SSL Certificate][2]

* Crear un [_proxy host_](instalacion-de-nginx-proxy-manager-en-docker#certificado-ssl) para el _hostname_ anterior: `vault.mydomain.com` &rarr; `http://vaultwarden:80`

![Proxy host][3]

# Configuración

Una vez en marcha, se puede acceder a Vaultwarden a través de `HTTPS` usando Nginx Proxy Manager (en este ejemplo `https://vault.mydomain.com/`) y comenzar la configuración.

El primer paso consiste en crear una nueva cuenta de usuario para acceder al _vault_ pulsando en el botón `Create Account`:

* Email Address: `vaultwarden@mydomain.com`
* Name: `Manel`
* Master Password: `*********`
* Re-type Master Password: `*********`
* Master Password Hint (optional): `Pista`

Si el usuario se crea correctamente, se podrá iniciar sesión con el mismo y ver la siguiente pantalla:

![Vaultwarden][4]

> **Nota**: Es aconsejable crear todos los usuarios que se necesiten en este momento y así se puede deshabilitar el registro de usuarios para aumentar la seguridad.

## Habilitar [Admin Panel](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-admin-page)

Para habilitar la [página de administración](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-admin-page) hay que realizar lo siguiente:

* Ejecutar el comando `openssl rand -base64 48` para generar un _token_ aleatorio:

```
U543Xx6Bhj+uiv9DhNDcAExM9Cef2C644CC7wwFGUh7Z4I6OvOEaYZaxjnrrKZYX
```

* Agregar la variable `ADMIN_TOKEN` al fichero `docker-compose.yml`:

```yaml
    environment:
      ADMIN_TOKEN: 'U543Xx6Bhj+uiv9DhNDcAExM9Cef2C644CC7wwFGUh7Z4I6OvOEaYZaxjnrrKZYX'
```

* Acceder al Vaultwarden Admin Panel a través de `https://vault.mydomain.com/admin`:

![Admin Panel][5]

> **Nota**: La primera vez que se guarde la configuración, se generará el fichero `/data/config.json`. Este fichero tiene preferencia sobre las variables indicadas en `docker-compose.yml`.

> **Nota**: También se podría haber activado el Admin Panel creando o modificando el fichero `/data/config.json` añadiendo la variable `admin_token` tal como se muestra a continuación:

```json
{
  "admin_token": "U543Xx6Bhj+uiv9DhNDcAExM9Cef2C644CC7wwFGUh7Z4I6OvOEaYZaxjnrrKZYX"
}
```

Algunos valores interesantes a configurar:

* General settings
  * Domain URL: `https://vault.mydomain.com/`
  * Trash auto-delete days: `90`
  * [Allow new signups](https://github.com/dani-garcia/vaultwarden/wiki/Disable-registration-of-new-users): `Disable`
  * Org creation users: `vaultwarden@mydomain.com`
  * [Allow invitations](https://github.com/dani-garcia/vaultwarden/wiki/Disable-invitations): `Disable`
* Advanced settings
* Yubikey settings
* Global Duo settings
* SMTP Email Settings
* Email 2FA Settings
* Read-Only Config
* Backup Database

> **Nota**: Una vez se haya configurado el entorno, es recomendable deshabilitar el Admin Panel. Para ello hay que eliminar la variable de entorno `ADMIN_TOKEN` y asegurarse que no exista la variable `admin_token` en el fichero `/data/config.json`.

# Actualizar

Si ya se había instalado Vaultwarden anteriormente, se puede actualizar de la siguiente manera:

```
docker stop vaultwarden
docker rm vaultwarden
docker rmi vaultwarden/server
docker-compose up -d
```

# Soporte

Algunos comandos para gestionar la configuración del contenedor:

```
# Acceder al shell mientras el contenedor está ejecutándose
docker exec -it vaultwarden /bin/bash

# Monitorizar los logs del contenedor en tiempo real
docker logs -f vaultwarden

# Obtener la versión de Vaultwarden
docker exec vaultwarden /vaultwarden --version
```

# Referencias

* [Vaultwarden Wiki](https://github.com/dani-garcia/vaultwarden/wiki)
  * [Docker Compose](https://github.com/dani-garcia/vaultwarden/wiki/Using-Docker-Compose)
  * [Private Vaultwarden + Let's Encrypt](https://github.com/dani-garcia/vaultwarden/wiki/Running-a-private-vaultwarden-instance-with-Let%27s-Encrypt-certs)
  * [WebSocket](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-WebSocket-notifications)  
* [Self Hosted Password Manager - Your data under your control!](https://www.youtube.com/watch?v=ub8jj96_Q3g&t=699s) @ YouTube "Christian Lempa"
* [¿Y si tú también estás cometiendo estos errores con tus contraseñas?](https://www.youtube.com/watch?v=U8dcl3jN-jg) @ YouTube "Un loco y su tecnología"
* [Vaultwarden en Unraid](https://firstcommit.dev/2021/05/09/vaultwarden/) @ First Commit
* [Bitwarden vs Vaultwarden](https://www.flopy.es/bitwarden-vs-vaultwarden-instalacion-y-uso-de-un-gestor-de-contrasenas-en-tu-propia-nube/) @ Flopy.es

[1]: /assets/img/blog/2022-11-13_image_1.png "DNS Type A"
[2]: /assets/img/blog/2022-11-13_image_2.png "SSL Certificate"
[3]: /assets/img/blog/2022-11-13_image_3.png "Proxy host"
[4]: /assets/img/blog/2022-11-13_image_4.png "Vaultwarden"
[5]: /assets/img/blog/2022-11-13_image_5.png "Admin Panel"
