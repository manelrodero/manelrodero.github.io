---
layout : post
blog-width: true
title: 'Instalación de Portainer CE en Docker'
date: '2021-04-01 23:53:01'
published: true
tags:
- Raspberry
- Docker
author:
  display_name: Manel Rodero
---

[Portainer CE](https://www.portainer.io/products/community-edition) es una herramienta de código abierto que ayuda a crear, administrar y mantener entornos de contenedores sin la necesidad de conocer comandos o sintaxis complejos.

Portainer está formado por dos elementos, **Portainer Server** y **Portainer Agent**, que se ejecutan en un contenedor Docker ligero. La instalación de Portainer CE en un [entorno Docker](https://documentation.portainer.io/v2.0/deploy/ceinstalldocker/) es bastante sencilla:

```
docker volume create portainer_data
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer-ce --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
```

# Configuración

Una vez instalado, Portainer es accesible mediante el puerto `9000`.

La primera vez que se accede a la aplicación hay que realizar una **configuración inicial** básica:

* Usuario administrador: `admin`
* Contraseña: `**********`
* No marcar la opción para recoger estadísticas anónimas
* Crear el usuario
* Conectar con el entorno local Docker

# docker-compose

La alternativa para instalar Portainer es utilizar `docker-compose`. Para ello se genera un fichero `docker-compose.yml` con el siguiente contenido:

```
  portainer:
    image: portainer/portainer-ce
    container_name: portainer-ce
    volumes:
    - /var/run/docker.sock:/var/run/docker.sock
    - ./volumes/data:/data
    ports:
    - 8000:8000
    - 9000:9000
    restart: always
```

Nótese que se ha utilizado un [**Host Volume**](https://www.digitalocean.com/community/tutorials/how-to-share-data-between-the-docker-container-and-the-host), es decir un directorio de la máquina _host_ donde quedará guardada la configuración si se destruye el contenedor.

Para iniciar el contenedor, únicamente hay que ejecutar `docker-compose up -d`.

# Actualizar

Si ya se había instalado Portainer anteriormente, se puede actualizar de la siguiente manera:

```
docker stop portainer-ce
docker rm portainer-ce
docker rmi portainer/portainer-ce
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer-ce --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
```

## Referencias

* [How To Share Data Between the Docker Container and the Host](https://www.digitalocean.com/community/tutorials/how-to-share-data-between-the-docker-container-and-the-host)
* [docker-compose up](https://docs.docker.com/compose/reference/up/)
* [Installing Portainer to the Raspberry Pi](https://pimylifeup.com/raspberry-pi-portainer/)
