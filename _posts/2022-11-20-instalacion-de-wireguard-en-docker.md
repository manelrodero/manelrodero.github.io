---
layout : post
blog-width: true
title: 'Instalación de WireGuard en Docker'
date: '2022-11-20 18:49:38'
published: true
tags:
- Docker
author:
  display_name: Manel Rodero
---

[WireGuard](https://www.wireguard.com/) es un nuevo protocolo VPN de código abierto que utiliza [criptografía de última generación](https://www.wireguard.com/protocol/) y que tiene como objetivo reemplazar a los protocolos VPN existentes como IPsec, [OpenVPN](https://openvpn.net/community/) o IKEv2.

La [instalación en Docker](https://hub.docker.com/r/linuxserver/wireguard) se realiza usando la imagen `linuxserver/wireguard`.

La forma más sencilla de hacerlo es usando un fichero [`docker-compose.yml`](https://github.com/linuxserver/docker-wireguard) con el siguiente contenido:

```yaml
version: "2.1"
services:
  wireguard:
    image: lscr.io/linuxserver/wireguard:latest
    container_name: wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Madrid
      - SERVERURL=wireguard.myhome.com # DNS dinámico
      - SERVERPORT=51820
      - PEERS=4 # 4 clientes
      - PEERDNS=9.9.9.9,149.112.112.112 # Quad9
      - INTERNAL_SUBNET=10.2.0.0/24 # Red WireGuard
      - ALLOWEDIPS=192.168.1.0/24 # Red que se enruta
      - LOG_CONFS=false # No guardar conf en logs
    volumes:
      - ~/volumes/wireguard:/config
      - /lib/modules:/lib/modules
    ports:
      - 51820:51820/udp
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
    restart: unless-stopped
```

> **Nota**: Dado que los contenedores Linux no tienen privilegios, es necesario añadir una serie de [_capabilities_](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) para permitir la gestión de la red (`NET_ADMIN`) y la carga/descarga de módulos del kernel en el _host_ (`SYS_MODULE`).

La variable `PEERS` indica el número de clientes para los cuales se crearán claves públicas/privadas y los ficheros de configuración.

También es posible utilizar un conjunto de _strings_ separados por comas para identificar a cada uno de los clientes (p.ej. `PEERS=me,you`).

Para iniciar el contenedor, se puede usar `docker-compose up -d` o usar el contenido del fichero anterior en **Portainer**.

# Configuración

Una vez en marcha, se puede visualizar el registro del servidor mediante el siguiente comando:

```
docker-compose logs -f wireguard
```

> **Nota**: Si se habilita `LOG_CONFS`, se podrán ver en el registro tantos códigos QR como _peer_ se hayan configurado en el fichero `docker-compose.yml`:

![QR de WireGuard][1]

Estos códigos QR permiten configurar un cliente (p.ej. [WireGuard para Android](https://play.google.com/store/apps/details?id=com.wireguard.android)) de una forma muy sencilla.

Si en algún momento se quieren recuperar estos códigos QR, únicamente hay que ejecutar la aplicación `show-peer` tal como se muestra a continuación:

```
docker exec -it wireguard /app/show-peer 1
docker exec -it wireguard /app/show-peer me
```

Para ver los **ficheros de configuración** se puede acceder a la _shell_ del contenedor usando el comando `docker exec`:

```
docker exec -it wireguard /bin/bash
```

y, a continuación, ir al directorio de configuración `/config` para verlos:

```
cd config
cd peer_me
cat peer_me.conf
```

El fichero de configuración contiene la información necesaria para conectarse a la red WireGuard y enrutar tráfico a través de ella:

```
[Interface]
Address = 10.2.0.2
PrivateKey = bC7ab/PmXsaMxxZZeF2rZlZcjSTvXmWKmQkkad9mzAA=
ListenPort = 51820
DNS = 9.9.9.9,149.112.112.112

[Peer]
PublicKey = aBBMYx2KclNaLTA3Biz8z105m3xGwQELlBjSjgfYzwi=
PresharedKey = 1v8IjmsC7+m5kJH/LJeMI7ZDKGO0GkNqesOlk+dATjD=
Endpoint = wireguard.myhome.com:51820
AllowedIPs = 192.168.1.0/24
```

> **Nota**: A la hora de configurar un cliente, el valor de `AllowedIPs` es muy importante. Si se deja el valor por defecto, `0.0.0.0/0`, se enrutará todo el tráfico del cliente a través de WireGuard.

# Configuración _router_

Antes de continuar con la configuración de los clientes, es necesario habilitar el **port forwarding** en el _router_ hacia el puerto `51820/UDP` del dispositivo donde esté el servidor WireGuard:

![Port Forwarding][2]

# Actualizar

Si ya se había instalado WireGuard anteriormente, se puede actualizar de la siguiente manera:

```
docker stop wireguard
docker rm vireguard
docker rmi lscr.io/linuxserver/wireguard
docker-compose up -d
```

# Soporte

Algunos comandos para gestionar la configuración del contenedor:

```
# Acceder al shell mientras el contenedor está ejecutándose
docker exec -it wireguard /bin/bash

# Monitorizar los logs del contenedor en tiempo real
docker logs -f wireguard
```

# Referencias

* [WireGuard para acceso seguro a Home Assistant y Node-RED](https://youtu.be/BoJ3wUZ4PJw) @ YouTube "Un loco y su tecnología"
* [Instalar y configurar el servidor VPN Wireguard con Docker](https://geekland.eu/instalar-y-configurar-el-servidor-vpn-wireguard-con-docker/)
* [WireGuard AllowedIPs Calculator](https://www.procustodibus.com/blog/2021/03/wireguard-allowedips-calculator/)  @ Pro Custodibus
* [WireGuard endpoints and IP addresses](https://www.procustodibus.com/blog/2021/01/wireguard-endpoints-and-ip-addresses/) @ Pro Custodibus
* [WireGuard installation and configuration - on Linux](https://www.youtube.com/watch?v=bVKNSf1p1d0) @ YouTube "Christian Lempa"
* [Create your own VPN server with WireGuard in Docker](https://www.youtube.com/watch?v=GZRTnP4lyuo) @ YouTube "Christian Lempa"

[1]: /assets/img/blog/2022-11-20_image_1.png "QR de WireGuard"
[2]: /assets/img/blog/2022-11-20_image_2.png "Port Forwarding"