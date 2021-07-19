---
layout : post
blog-width: true
title: 'Instalar Docker en Raspberry Pi 4'
date: '2021-02-27 16:00:00'
published: true
tags:
- Raspberry
author:
  display_name: Manel Rodero
---

Instalar [Docker](https://www.docker.com/) en una Raspberry Pi 4 es bastante fácil porque [Raspberry Pi OS](https://docs.docker.com/engine/install/debian/#install-using-the-convenience-script) está soportado oficialmente.

{: .box-note}
**Nota**: Es necesario actualizar el sistema operativo antes de continuar: `sudo apt update` y `sudo apt upgrade -y`.

La descarga del script de instalación se realiza mediante el siguiente comando:

```
curl -fsSL https://get.docker.com -o get-docker.sh
```

A continuación se pueden ejecutar los siguientes comandos para realizar la instalación:

```
chmod u+x get-docker.sh
sudo ./get-docker.sh
```

Para que un usuario que no es `root` pueda ejecutar _Docker_ correctamente hay que añadirlo al grupo correspondiente (hay que tener en cuenta posibles [problemas de seguridad](https://docs.docker.com/engine/security/security/#docker-daemon-attack-surface)):

```
sudo usermod -aG docker $(whoami)
```

Finalmente, se reinicia:

```
sudo reboot
```

Y cuando haya reiniciado, se puede comprobar la versión mediante el comando `docker version`. En el momento de escribir este artículo la última versión disponible es la 20.10.3:

```
docker version
```
```
Client: Docker Engine - Community
 Version:           20.10.4
 API version:       1.41
 Go version:        go1.13.15
 Git commit:        d3cb89e
 Built:             Thu Feb 25 07:06:32 2021
 OS/Arch:           linux/arm
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.4
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.13.15
  Git commit:       363e9a8
  Built:            Thu Feb 25 07:04:06 2021
  OS/Arch:          linux/arm
  Experimental:     false
 containerd:
  Version:          1.4.3
  GitCommit:        269548fa27e0089a8b8278fc4fc781d7f65a939b
 runc:
  Version:          1.0.0-rc92
  GitCommit:        ff819c7e9184c13b7c2607fe6c30ae19403a7aff
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

## Instalar Docker Compose

[Docker Compose](https://docs.docker.com/compose/) es una herramienta para definir y ejecutar aplicaciones Docker con múltiples ontenedores.

Uno de los métodos más sencillos para instalarla es usando `pip`. Si no se dispone de este gestor de paquetes Python, se puede instalar de la siguiente manera:

```
# Dependencias
sudo apt install libffi-dev libssl-dev -y

# Instalación del gestor de paquetes 'pip'
sudo apt install python3-pip -y
```

A continuación se instala Docker Compose:

```
sudo pip3 install docker-compose
```
 
Si todo funciona correctamente, se puede comprobar la versión:

```
docker-compose version
```
```
docker-compose version 1.28.5, build unknown
docker-py version: 4.4.4
CPython version: 3.7.3
OpenSSL version: OpenSSL 1.1.1d  10 Sep 2019
```

## Pruebas

Para comprobar que todo funciona correctamente se puede ejecutar una aplicación simple como `hello-world`:

```
docker search hello-world
docker run hello-world
```

## Referencias

* [How to Install Docker on Raspberry Pi 4](https://linuxhint.com/install_docker_raspberry_pi-2/)
* [Docker y Raspberry. Instalar Docker y Docker Compose](https://www.atareao.es/como/docker-y-raspberry-instalar-docker-y-docker-compose/)
* [The Best Docker Setup and Docker Guide](https://wiki.servarr.com/Docker_Guide)

<p></p>
