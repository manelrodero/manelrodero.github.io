---
layout : post
blog-width: true
title: 'Instalación de Portainer CE en Docker'
date: '2021-04-01 23:53:01'
published: true
tags:
- Docker
author:
  display_name: Manel Rodero
---

#### _**Actualizaciones**:_

* **2022-10-22**: Se cambia el puerto 9000 por el 9443 ya que ahora Portainer CE utiliza HTTPS.
* **2022-12-03**: Se actualizan algunos enlaces a recursos que han cambiado su ubicación o ya no están disponibles.

# Instalación

[**Portainer CE**](https://github.com/portainer/portainer) es una herramienta de código abierto que ayuda a crear, administrar y mantener entornos de contenedores sin la necesidad de conocer comandos o sintaxis complejas.

Portainer está formado por dos elementos, **Portainer Server** y **Portainer Agent**, que se ejecutan en un contenedor Docker ligero.

La instalación de Portainer CE en un [entorno Docker](https://docs.portainer.io/start/install/server/docker) es bastante sencilla usando el comando [`docker run`](https://docs.docker.com/engine/reference/commandline/run/):

```
docker run -d \
  -p 9443:9443 \
  --name=portainer-ce \
  --restart=unless-stopped \
  -v /etc/localtime:/etc/localtime:ro \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v ~/volumes/portainer-ce:/data \
  portainer/portainer-ce:latest
```

> **Nota**: También será necesario exponer un servidor de túnel TCP a través del puerto 8000 (`-p 8000:8000`) si se planea utilizar las funciones de [**Edge Compute**](https://docs.portainer.io/admin/settings/edge).

# Configuración

Una vez en marcha, se puede acceder a Portainer a través del puerto `9443` (en este ejemplo [https://192.168.1.180:9443](https://192.168.1.180:94443)) y realizar una **configuración inicial** básica:

* Usuario administrador: `admin`
* Contraseña: `**********`
* No marcar la opción para recoger estadísticas anónimas
* Crear el usuario
* Conectar con el entorno local Docker

> **Nota**: Portainer genera y utiliza un certificado SSL autofirmado para proteger el puerto 9443. Podemos proporcionar nuestro propio certificado SSL durante la [instalación](https://docs.portainer.io/advanced/ssl#docker-standalone) o a través de la [interfaz de usuario](https://docs.portainer.io/admin/settings#ssl-certificate) una vez completada la instalación.

Una vez hecho ésto, se puede leer la documentación "[Using Portainer](https://docs.portainer.io/user/home)" para conocer todo lo que puede hacer con esta aplicación:

* [Stacks](https://docs.portainer.io/user/docker/stacks)
* [Networks](https://docs.portainer.io/user/docker/networks)
* [Volumes](https://docs.portainer.io/user/docker/volumes)
* [Secrets](https://docs.portainer.io/user/docker/secrets)
* etc.

# Usando `docker-compose`

Otra alternativa para instalar Portainer es utilizar el comando `docker-compose`. Para ello se genera un fichero `docker-compose.yml` con el siguiente contenido:

```yaml
version: '3'
services:
  portainer-ce:
    image: portainer/portainer-ce:latest
    container_name: portainer-ce
    volumes:
    - /etc/localtime:/etc/localtime:ro
    - /var/run/docker.sock:/var/run/docker.sock
    - ~/volumes/portainer-ce:/data
    ports:
    - 9443:9443
    restart: unless-stopped
```

> **Nota**: En la configuración anterior se ha utilizado un [**Host Volume**](https://www.digitalocean.com/community/tutorials/how-to-share-data-between-the-docker-container-and-the-host), es decir, un directorio de la máquina _host_ donde quedará guardada la configuración de Portainer si se destruye el contenedor.

Para iniciar el contenedor, únicamente hay que ejecutar el comando `docker-compose up -d`.

# Actualizar

> **Nota**: Antes de actualizar Portainer, es recomendable descargar una [copia de seguridad](https://docs.portainer.io/admin/settings#backup-portainer) de la configuración accediendo a `Settings` &rarr; `Backup Portainer` &rarr; `Download backup file`.

Si ya se había instalado Portainer anteriormente, se puede actualizar de la siguiente manera:

```
cd ~/dockers/portainer-ce
docker stop portainer-ce
docker rm portainer-ce
docker rmi portainer/portainer-ce
docker-compose up -d
```

# Soporte

Algunos comandos para gestionar la configuración del contenedor:

```
# Monitorizar los logs del contenedor en tiempo real
docker logs -f portainer-ce
```

## Referencias

* [Installing Portainer to the Raspberry Pi](https://pimylifeup.com/raspberry-pi-portainer/)
* [Deploying Portainer CE in Docker](https://documentation.portainer.io/v2.0/deploy/ceinstalldocker/)
* [How To Share Data Between the Docker Container and the Host](https://www.digitalocean.com/community/tutorials/how-to-share-data-between-the-docker-container-and-the-host)
* [The Best Docker Setup: Consistent and well planned paths](https://wiki.servarr.com/docker-guide#consistent-and-well-planned-paths)
* [Stacks = docker-compose, the Portainer way](https://www.portainer.io/blog/stacks-docker-compose-the-portainer-way)
* [Ultimate Media Server : Episode 1 - OpenMediaVault, Docker & Portainer](https://youtu.be/ZLa5NGPKQv0?list=PLhMI0SExGwfAdXDmYJ9jt_SxjkEfcUwEB)
