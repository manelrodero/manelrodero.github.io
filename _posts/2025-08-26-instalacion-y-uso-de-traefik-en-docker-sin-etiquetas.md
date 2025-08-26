---
layout : post
blog-width: true
title: 'Instalación y uso de Traefik en Docker sin etiquetas'
date: '2025-08-26 07:18:55'
#last-updated: '2025-08-26 07:18:55'
published: true
tags:
- Docker
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2025-08-26_cover.png"
thumbnail-img: ""
---

El próximo mes de noviembre se cumplirán tres años desde que [instalé Nginx Proxy Manager en Docker](instalacion-de-nginx-proxy-manager-en-docker) por primera vez en mi entorno de `#HomeLab`.

Aunque estoy bastante contento con su facilidad de uso y comodidad, cuando comienzas a tener servidores y servicios se necesita algo más moderno, dinámico y automatizable.

Por este motivo he decidido instalar [**Traefik**](https://doc.traefik.io/traefik/){:target="_blank"} usando Docker pero, a diferencia de la mayoría de configuraciones, **no usaré etiquetas** para realizar el enrutado de los servicios.

He usado la misma táctica que mi compañero de trabajo Robert y he usado **ficheros** para dividir la configuración de Traefik en dos categorías principales:

* **configuración de instalación** (o configuración estática). Aquella configuración que requiere reiniciar Traefik para aplicarla (_entry points_, _providers_, etc.)
* **configuración de enrutado** (o configuración dinámica). Incluye elementos que pueden ser actualizados sin reiniciar Traefik (_routers_, _services_ y _middlewares_)

## Instalación en Docker (`compose.yaml`)

La instalación en [Docker](https://doc.traefik.io/traefik/getting-started/docker/){:target="_blank"} es bastante sencilla y únicamente requiere definir un fichero `compose.yaml` con el siguiente contenido:

```yaml
services:
  traefik:
    image: traefik:v3.5.0
    container_name: traefik
    hostname: traefik

    restart: unless-stopped
    security_opt:
      - no-new-privileges:true

    environment:
      TZ: ${TZ:-Europe/Madrid}
      CF_DNS_API_TOKEN: ${CF_DNS_API_TOKEN}

    networks:
      - traefik
    ports:
      - "80:80"
      - "443:443"

    volumes:
      - /usr/share/zoneinfo/${TZ:-Europe/Madrid}:/etc/localtime:ro
      - ./data/traefik.yaml:/traefik.yaml:ro
      - ./data/conf.d:/conf.d:ro
      - ./data/ssl:/ssl
      - ./data/logs:/var/log/traefik

    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

networks:
  traefik:
    name: traefik
```

Para "completar" el entorno se necesitan crear algunos ficheros y directorios:

```bash
touch .env
mkdir -p ./data/conf.d ./data/ssl ./data/logs
touch ./data/ssl/acme.json
chmod 600 ./data/ssl/acme.json
```

En el fichero `.env` se definirá, como mínimo, la variable `CF_DNS_API_TOKEN` con un **token** de Cloudflare que permita editar nuestra zona DNS.

```plaintext
TZ=Europe/Madrid
CF_DNS_API_TOKEN=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

## Configuración **estática** (`traefik.yaml`)

El fichero `./data/traefik.yaml` contendrá la [configuración **estática**](https://doc.traefik.io/traefik/reference/install-configuration/configuration-options/){:target="_blank"} de Traefik:

```yaml
################################################################
# Global configuration
################################################################

global:
  checknewversion: false
  sendanonymoususage: false

################################################################
# API and dashboard configuration
################################################################

api:
  insecure: false
  dashboard: true
  debug: false

################################################################
# Load dynamic configuration from .yaml files in a directory
# - Routers
# - Middleware
# - Services
################################################################

providers:
  file:
    directory: /conf.d
    watch: true

################################################################
# Certificate Resolvers
################################################################

certificatesResolvers:
  letsencrypt:
    acme:
      email: acme@midominio.com
      storage: /ssl/acme.json
      caServer: https://acme-v02.api.letsencrypt.org/directory
      dnsChallenge:
        provider: cloudflare
#        resolvers:
#          - "1.1.1.1:53"
#          - "1.0.0.1:53"

  letsencrypt_staging:
    acme:
      email: acme@midominio.com
      storage: /ssl/acme.json
      caServer: https://acme-staging-v02.api.letsencrypt.org/directory
      dnsChallenge:
        provider: cloudflare
#        resolvers:
#          - "1.1.1.1:53"
#          - "1.0.0.1:53"

################################################################
# EntryPoints configuration
################################################################

entryPoints:
  web:
    address: ":80"
    http:
      redirections:
        entryPoint:
          to: websecure
          scheme: https
  websecure:
    address: ":443"
    http:
      tls:
#        certResolver: letsencrypt
        certResolver: letsencrypt_staging

################################################################
# Traefik logs configuration
# - common: formato plano (para humanos)
################################################################

log:
  filePath: "/var/log/traefik/traefik.log"
  format: "common"
#  format: "json"
#  level: INFO
  level: DEBUG

################################################################
# Access logs configuration
# - common: Apache/Nginx
# - combined: common + otros campos (Referer, User-Agent, etc.)
# Con los formatos anteriores, no hay filtros/fields/headers
################################################################

accessLog:
  filePath: "/var/log/traefik/traefik-access.log"
  format: "json"
  filters:
    statusCodes:
      - "200"
      - "400-599"
    retryAttempts: true
    minDuration: "10ms"
  bufferingSize: 0
  fields:
    headers:
      defaultMode: drop
      names:
        User-Agent: keep
```

{: .box-note}
**Nota**: Para evitar problemas con los [límites del entorno de producción](https://letsencrypt.org/docs/rate-limits/) de Let's Encrypt, se recomienda usar el [entorno de **staging**](https://letsencrypt.org/docs/staging-environment/){:target="_blank"} durante las pruebas iniciales.

## Configuración **dinámica**

El fichero o ficheros con la [configuración **dinámica**](https://doc.traefik.io/traefik/reference/routing-configuration/dynamic-configuration-methods/){:target="_blank"} se dejarán en el directorio `./data/conf.d`.

Este directorio está vigilado constantemente por Traefik y, en cuanto se escriba un fichero de configuración válido, se cargará y se aplicará.

Aunque Traefik permite hacer cosas bastante complejas con los [`routers`](https://doc.traefik.io/traefik/reference/routing-configuration/http/router/rules-and-priority/){:target="_blank"}, [`services`](https://doc.traefik.io/traefik/reference/routing-configuration/http/load-balancing/service/){:target="_blank"} y [`middlewares`](https://doc.traefik.io/traefik/reference/routing-configuration/http/middlewares/overview/){:target="_blank"}, en mi caso usaré básicamente los enrutados:

* `HTTP` &rarr; `HTTPS`
* `HTTPS` &rarr; `HTTPS`

Un ejemplo de fichero de configuración para el primer tipo de servicio es el siguiente:

```yaml
# Fichero homeassistant.yaml
http:
  routers:
    homeassistant:
      rule: "Host(`homeassistant.midominio.com`)"
      service: homeassistant
      entryPoints:
        - websecure
      tls:
        certResolver: letsencrypt

  services:
    homeassistant:
      loadBalancer:
        servers:
          - url: "http://<ip-homeassistant>:8123"
```

Un ejemplo de fichero de configuración para el segundo tipo de servicio (que normalmente tiene certificados auto-firmados que no queremos comprobar) es el siguiente:

```yaml
# Fichero proxmox.yaml
http:
  routers:
    proxmox:
      rule: "Host(`proxmox.midominio.com`)"
      service: proxmox
      entryPoints:
        - websecure
      tls:
        certResolver: letsencrypt

  serversTransports:
    proxmox:
      insecureSkipVerify: true

  services:
    proxmox:
      loadBalancer:
        servers:
          - url: "https://<ip-proxmox>:8006"
        serversTransport: proxmox
        passHostHeader: false
```

Se puede usar el siguiente _script_ plantilla para pegarlo en la _shell_ y crear un fichero de configuración de forma rápida:

```bash
service="homeassistant"
url="http://<ip-homeassistant>:8123"

cat << EOF > "${service}.yaml"
http:
  routers:
    ${service}:
      rule: "Host(\`${service}.midominio.com\`)"
      service: ${service}
      entryPoints:
        - websecure
      tls:
        certResolver: letsencrypt

  services:
    ${service}:
      loadBalancer:
        servers:
          - url: "${url}"
EOF
```

## Certificados `acme.json`

El fichero `./data/ssl/acme.json` contendrá los **certificados** obtenidos desde [Let's Encrypt](https://letsencrypt.org/){:target="_blank"} usando un [**DNS Challenge**](https://doc.traefik.io/traefik/user-guides/docker-compose/acme-dns/){:target="_blank"} contra [Cloudflare](https://dash.cloudflare.com/){:target="_blank"}.

## Prueba

Si la configuración es correcta, al acceder a un servicio a través de Traefik se debería realizar el **enrutado** y usar un **certificado** válido de Let's Encrypt tal como se muestra a continuación:

![Certificado Let's Encrypt][1]

{: .box-note}
**Nota**: Aunque no lo he comentado, es necesario crear entradas para cada servicio en el DNS apuntando a la IP de la máquina en la que se haya instalado Traefik ;-)

## Protección del _dashboard_

El _dashboard_ de Traefik se puede proteger con **autenticación básica** usando el siguiente fichero de configuración dinámica:

```yaml
# Fichero traefik.yaml
http:
  middlewares:
    traefik:
      basicAuth:
        users:
          - "admin:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

  routers:
    traefik:
      rule: "Host(`traefik.midominio.com`)"
      service: api@internal
      entryPoints:
        - websecure
      middlewares:
        - traefik
      tls:
        certResolver: letsencrypt
```

Y generando el password del usuario `admin` mediante la siguiente ejecución del comando `htpasswd` de las utilidades de Apache:

```bash
docker run -it --rm httpd:2.4 /bin/sh
htpasswd -B -n admin
```

## Referencias

* [Traefik Proxy Documentation](https://doc.traefik.io/traefik/){:target="_blank"}
* [The Ultimate Guide to Setting Up Traefik](https://medium.com/@svenvanginkel/the-ultimate-guide-to-setting-up-traefik-650bd68ae633){:target="_blank"}
* [Introduction to Traefik](https://www.baeldung.com/traefik-tutorial){:target="_blank"}
* [Traefik v2 Examples](https://github.com/DoTheEvo/Traefik-v2-examples){:target="_blank"}
* [Configuring Traefik to work over Taiscale](https://binarypatrick.dev/posts/traefik-over-tailscale/){:target="_blank"}
* [Traefik Let's Encrypt Certificates Configuration](https://www.virtualizationhowto.com/2023/02/traefik-letsencrypt-certificates-configuration/){:target="_blank"}

### Historial de cambios

* **2025-08-26**: Documento inicial

[1]: /assets/img/blog/2025-08-26_image_1.png "Certificado Let's Encrypt"
