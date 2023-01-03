---
layout : post
blog-width: true
title: 'DNS dinámico gratuito usando Cloudflare'
date: '2022-12-17 13:47:19'
published: true
tags:
- HomeLab
author:
  display_name: Manel Rodero
---

Es posible utilizar [Cloudflare](https://www.cloudflare.com/) de forma gratuita para proyectos o _hobbies_ de carácter **personal**.

# Registro

Registrarse en Cloudflare es un proceso muy sencillo:

* Acceder a [`https://dash.cloudflare.com/sign-up`](https://dash.cloudflare.com/sign-up)
* Rellenar el formulario:
  * Dirección de correo electrónico: `usuario@midominio.com`
  * Contraseña: `*********`
  * No marcar la opción para recibir correos ocasionales
* Pulsar el botón `Sign Up`

Una vez hecho ésto, se recibirán dos correos electrónicos:

* _Welcome to Cloudflare Access_
* _[Cloudflare]: Please verify your email address_

Hay que seguir el enlace del segundo mensaje para verificar el correo electrónico y poder acceder al Dashboard de Cloudflare en [`https://dash.cloudflare.com/`](https://dash.cloudflare.com/).

# Cloudflare DNS

Para tener DNS dinámico, lo primero es añadir una _website_ o **dominio** a Cloudflare:

* Acceder a `Websites` en el menú de la izquierda
* Pulsar el botón `(+) Add Site`
* Indicar el nombre del sitio/dominio: `midominio.com`
* Pulsar el botón `Add site`

En el paso 1 del asistente hay que seleccionar el [**plan**](https://www.cloudflare.com/plans/) gratuito `Free $0/month` que tiene las siguientes características:

  * DNS rápido y fácil de usar
  * Protección DDoS no medida
  * CDN global
  * Certificado SSL Universal
  * Conjunto de reglas administradas gratuitas
  * Mitigación sencilla de bots
  * 3 Reglas de página
  * 5 Reglas [WAF](https://www.cloudflare.com/waf/)

En el paso 2, se obtienen los registros DNS actuales del dominio que se ha indicado y se muestran en pantalla para poder revisarlos:

![Registros DNS][2]

> **Nota**: Aquí se podría editar o borrar los registros existentes así como añadir nuevos registros (por ejemplo `vault.midominio.com` para acceder a una instalación de [Vaultwarden](instalacion-de-vaultwarden-en-docker) local).

* Una vez revisados, se pulsa el botón `Continue`

En el paso 3, y último, se dan las instrucciones para cambiar los **_nameservers_** desde el registrador actual a Cloudflare:

* Determinar el [registrador actual](https://lookup.icann.org/en)
* Acceder al registrador con una cuenta de administración
* Borrar los servidores de nombre actuales
* Introducir los servidores de nombres de Cloudflare que se indican en el asistente, por ejemplo:
  * `romina.ns.cloudflare.com`
  * `vick.ns.cloudflare.com`

* Pulsar el botón `Done, check nameservers`

Finalmente, se podrían configurar los parámetros del dominio para aumentar la seguridad, optimizar el rendimiento, etc. pero no es necesario hacerlo en este momento por lo que se pulsará la opción `Finish later`:

* Improve security
  * Automatic HTTPS Rewrites
  * Always use HTTPS
* Optimize performance
  * [Auto Minify](https://support.cloudflare.com/hc/articles/200169876)
  * [Brotli](https://support.cloudflare.com/hc/articles/200168396) (compresión)

> **Nota**: Los registradores pueden tardar hasta 24h en procesar los cambios. Hay que ser paciente y esperar antes de continuar con cualquier proceso que requiera del DNS en Cloudflare (por ejemplo, crear un certificado SSL de Let's Encrypt desde [Nginx Proxy Manager](instalacion-de-nginx-proxy-manager-en-docker)).

# DNS dinámico

Se puede usar Cloudflare para tener DNS dinámico al estilo de [Duck DNS](https://www.duckdns.org/) o [No-IP](https://www.noip.com/).

El uso habitual de este servicio sería tener un dominio, por ejemplo `casa.midominio.com`, que apunte a la IP pública que proporciona el proveedor de Internet.

La idea es crear un registro de tipo A y después usar alguna opción para que se actualice automáticamente, por ejemplo:

* [Integración de Cloudflare](https://www.home-assistant.io/integrations/cloudflare/) en Home Assistant
* [Cloudflare DDNS](https://github.com/timothymiller/cloudflare-ddns) by Timothy Miller
* [Docker CloudFlare DDNS](https://github.com/oznu/docker-cloudflare-ddns) by Oznu
* [Cloudflare Dynamic DNS IP Updater](https://github.com/K0p1-Git/cloudflare-ddns-updater) by K0p1-Git
* Etc.

## API Token

Para poder actualizar los registros DNS de forma automatizada, es necesario crear un **token** que asigne los permisos necesarios para hacerlo:

* Acceder a `My Profile` &rarr; `API Tokens`
* Pulsar el botón `Create Token`
* Seleccionar la plantilla `Edit Zone DNS` pulsando el botón `Use template`

> **Nota**: Esta plantilla agrega únicamente el permiso `Zone > DNS > Edit` por lo que será necesario añadir uno más.

* Agregar un nuevo permiso `Zone > Zone > Read`
* Seleccionar la zona que se incluirá: `Zone > Specific zone > midominio.com`
* Pulsar el botón `Continue to summary`
* Finalmente, pulsar el botón `Create Token`

Este proceso creará un _token_ similar al mostrado a continuación que se deberá copiar en algún lugar seguro ya que únicamente se muestra una vez:

```
8-3x-wz_B7cZXjMNE3Bz1bLMa4YsYQ-XESthjXe3
```

## Docker de Timothy Miller

La implementación _docker_ de [Timothy Miller](https://github.com/timothymiller/cloudflare-ddns) no funciona correctamente en mi entorno.

Tal como se indica en la [issue 111](https://github.com/timothymiller/cloudflare-ddns/issues/111) del proyecto, ésto es debido al uso de [`1.1.1.1`](https://1.1.1.1/cdn-cgi/trace) en lugar de [`cloudflare.com`](https://cloudflare.com/cdn-cgi/trace) para obtener la IP pública.

Una posible "solución", sería descargar el fichero [`cloudflare-ddns.py`](https://github.com/timothymiller/cloudflare-ddns/blob/master/cloudflare-ddns.py), revertir el cambio y ejecutarlo cada 5 minutos usando **crontab**.

El fichero de configuración `config.json` que se debería usar sería el siguiente:

```json
{
  "cloudflare": [
    {
      "authentication": {
        "api_token": "8-3x-wz_B7cZXjMNE3Bz1bLMa4YsYQ-XESthjXe3"
      },
      "zone_id": "a4eb0ce4b412b3d2ae53a0e0ab22502a",
      "subdomains": [
        {
          "name": "casa",
          "proxied": false
        }
      ]
    }
  ],
  "a": true,
  "aaaa": false,
  "purgeUnknownRecords": false,
  "ttl": 300
}
```

## Docker de Oznu

La implementación _docker_ de [Oznu](https://github.com/oznu/docker-cloudflare-ddns) es bastante sencilla y funciona correctamente usando un fichero `docker-compose.yml` similar al siguiente:

```yaml
version: "2"
services:
  cloudflare-ddns:
    image: oznu/cloudflare-ddns:latest
    container_name: cloudflare-ddns
    environment:
      - API_KEY=8-3x-wz_B7cZXjMNE3Bz1bLMa4YsYQ-XESthjXe3
      - ZONE=midominio.com
      - SUBDOMAIN=casa
      - PROXIED=false
      - RRTYPE=A
      - DELETE_ON_STOP=false
      - DNS_SERVER=9.9.9.9
    restart: unless-stopped
```

## Script de K0p1-Git

El script de [K0p1-Git](https://github.com/K0p1-Git/cloudflare-ddns-updater) es bastante sencillo y funciona correctamente usando una configuración similar a la siguiente:

```bash
auth_email=""
auth_method="token"
auth_key="8-3x-wz_B7cZXjMNE3Bz1bLMa4YsYQ-XESthjXe3"
zone_identifier="a4eb0ce4b412b3d2ae53a0e0ab22502a"
record_name="casa.midominio.com"
ttl="300"
proxy="false"
```

# Túneles

Los túneles se gestionan desde un Dashboard diferente: [`https://one.dash.cloudflare.com/`](https://one.dash.cloudflare.com/) y se suelen usar cuando no se puede, o no se quiere, abrir puertos en el router por estar detrás de un CGNAT.

> **Nota**: Si se utiliza un túnel hay que **extremar la seguridad** para no dejar nada abierto y que los ciberdelincuentes entren hasta la cocina.

Si se quiere acceder a [Home Assistant](instalacion-de-home-assistant-en-docker), es mucho más recomendable pagar una subscripción a [Nabu Casa](https://www.nabucasa.com/) y evitar los problemas de seguridad que pueden tener los túneles.

De todos modos, lo mejor es **conectarse a la red local** [usando WireGuard como VPN](instalacion-de-wireguard-en-docker). De esta manera, se puede acceder a los servicios usando la dirección IP privada de cada servicio.

# Referencias

* [Dominio Propio y CloudFlare: La mejor alternativa a DuckDNS](https://www.youtube.com/watch?v=kKwIIR-jylQ) @ YouTube "Un loco y su tecnología"
* [DDNS on a Raspberry Pi using the Cloudflare API (Dynamic DNS)](https://www.youtube.com/watch?v=rI-XxnyWFnM) @ YouTube "NetworkChuck"
* [Get started with Cloudflare](https://developers.cloudflare.com/learning-paths/get-started/)
* [Cloudflare DNS > Zone setups > Full setup](https://developers.cloudflare.com/dns/zone-setups/full-setup)

[2]: /assets/img/blog/2022-12-17_image_2.png "Registros DNS"
