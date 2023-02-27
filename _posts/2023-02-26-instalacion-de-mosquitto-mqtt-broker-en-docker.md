---
layout : post
blog-width: true
title: 'Instalación de Mosquitto (MQTT Broker) en Docker'
date: '2023-02-26 13:47:53'
published: true
tags:
- Home Assistant
- Docker
author:
  display_name: Manel Rodero
---

# Introducción

Eclipse [Mosquitto](https://www.mosquitto.org/) es un _broker_ de mensajes de código abierto y tan liviano que se puede ejecutar sin problemas en una Raspberry Pi.

Mosquitto implementa el protocolo de mensajería simple **MQTT** (_Message Queuing Telemetry Transport_) diseñado para dispositivos limitados y con poco ancho de banda (p.ej. dispositivos IoT).

La comunicación MQTT funciona como un sistema de publicación y suscripción. Los dispositivos (p.ej. un sensor de temperatura) publican mensajes sobre un tema específico. Estos mensajes son recibidos por el _broker_ MQTT, que se encarga de filtrarlos, decidir quién está interesado en ellos y luego publicarlos para todos los clientes suscritos a ese tema.

![MQTT Broker (c) Cedalo][1]

# Instalación

La [instalación en Docker](https://hub.docker.com/_/eclipse-mosquitto) se realiza usando la imagen `eclipse-mosquitto`.

La forma más sencilla es usar un fichero `docker-compose.yml` con el siguiente contenido:

```yaml
version: "3"
services:
  mosquitto:
    image: eclipse-mosquitto:latest
    container_name: mosquitto
    environment:
      - TZ=Europe/Madrid    
    volumes:
      - /home/pi/volumes/mosquitto/config:/mosquitto/config
      - /home/pi/volumes/mosquitto/data:/mosquitto/data
      - /home/pi/volumes/mosquitto/log:/mosquitto/log
    ports:
      - 1883:1883
      - 9001:9001
    restart: unless-stopped
```

Se puede usar `docker-compose up -d` o usar el contenido del fichero en Portainer.

> **Nota**: Antes de ponerlo en marcha es necesario crear un fichero de configuración [`conf/mosquitto.conf`](https://mosquitto.org/man/mosquitto-conf-5.html) con el siguiente contenido:

```
persistence true
persistence_location /mosquitto/data/
log_dest file /mosquitto/log/mosquitto.log
listener 1883
socket_domain ipv4
# Mosquitto >= 2.0 únicamente permite conexiones autenticadas mediante usuario/contraseña
# Permitimos temporalmente las conexiones anónimas para probar el entorno
allow_anonymous true
```

A continuación, se puede usar el comando `tail` para ver si Mosquitto se ha iniciado correctamente:

```
pi@pi4nas: $ sudo tail -f /home/pi/volumes/mosquitto/log/mosquitto.log
1677440889: mosquitto version 2.0.15 starting
1677440889: Config loaded from /mosquitto/config/mosquitto.conf.
1677440889: Opening ipv4 listen socket on port 1883.
1677440889: mosquitto version 2.0.15 running
```

Finalmente, para probar el envío y recepción de mensajes, se crea una suscripción a los mensajes de un tema determinado usando el comando `mosquitto_sub`:

```
# Suscripción al tema /test/message
docker exec -it mosquitto mosquitto_sub -v -t /test/message
```

Y, desde otro terminal, se envían mensajes usando el comando `mosquitto_pub` al tema `/test/message`:

```
# Envío de mensajes al tema /test/message
docker exec -it mosquitto mosquitto_pub -t /test/message -m 'Hello World!'
docker exec -it mosquitto mosquitto_pub -t /test/message -m 'Hello World again!'
```

Si todo funciona correctamente, los mensajes se mostrarán en el terminal donde se había creado la suscripción:

```
pi@pi4nas:~ $ docker exec -it mosquitto mosquitto_sub -v -t /test/message
/test/message Hello World!
/test/message Hello World again!
```

# Configuración

Aunque durante las pruebas se ha utilizado conexiones anónimas, lo más recomendable es realizarlas de manera autenticada. Para ello es necesario crear un fichero de passwords [`conf/mosquitto.passwd`](https://mosquitto.org/man/mosquitto_passwd-1.html) usando el comando `mosquitto_passwd`:

```
pi@pi4nas: $ docker exec -it mosquitto mosquitto_passwd -c /mosquitto/config/mosquitto.passwd admin
Password: ********
Reenter password: ********
```

El contenido del fichero debería ser similar al siguiente, una entrada para el usuario `admin` seguida por `:` y el _hash_ `sha512-pbkdf2` de la contraseña introducida:

```
pi@pi4nas:~ $ cat /home/pi/volumes/mosquitto/config/mosquitto.passwd
admin:$6$utCGq9HKlEIB2HWO$Q77ea1FcXR9b1XoGU/iWQ1Yf9ptVZOgjW/gIxM/YECHdDECZrt0GONub3JtlkC0IGqFrZg4JemcqHzjc1QakQg==
```

A continuación se deberá modificar el fichero de configuración y reiniciar el _docker_:

```
allow_anonymous false
password_file /mosquitto/config/mosquitto.passwd
```

Si se intenta realizar la misma prueba de suscripción/envío realizada anteriormente debería fallar:

```
pi@pi4nas:~ $ docker exec -it mosquitto mosquitto_sub -v -t /test/message
Connection error: Connection Refused: not authorised.
```

Para que todo vuelva a funcionar correctamente se deberá especificar el usuario y contraseña:

```
# Suscripción
docker exec -it mosquitto mosquitto_sub -u admin --pw password -v -t /test/message
# Envío
docker exec -it mosquitto mosquitto_pub -u admin --pw password -t /test/message -m 'Hello World!'
```

Y los _logs_ deberían mostrar la información de conexión:

```
# Conexión del suscriptor (se mantiene escuchando)
1677522832: New connection from 127.0.0.1:42796 on port 1883.
1677522832: New client connected from 127.0.0.1:42796 as auto-CF9D1353-ACA4-247E-AEE2-200CC336F081 (p2, c1, k60, u'admin').

# Conexión del publicador (envía y desconecta)
1677522863: New connection from 127.0.0.1:42876 on port 1883.
1677522863: New client connected from 127.0.0.1:42876 as auto-C9E3C15F-94B6-BE9E-6AE0-9B99CBC27580 (p2, c1, k60, u'admin').
1677522863: Client auto-C9E3C15F-94B6-BE9E-6AE0-9B99CBC27580 disconnected.
```

# Integración con Home Assistant

La integración con [Home Assistant](instalacion-de-home-assistant-en-docker) es muy sencilla:

* Acceder a `Settings`
* Acceder a `Device and Services`
* Pulsar en el botón `ADD INTEGRATION`
* Buscar la integración **MQTT** y añadirla
* En el _wizard_ de configuración se introduce la siguiente información:
  * Broker: `127.0.0.1`
  * Port: `1883`
  * Username: `admin`
  * Password: `********`
* Pulsar en `SUBMIT`

A continuación se puede realizar la misma prueba de suscripción a un _topic_ desde la configuración de la integración:

* Configure MQTT
* Listen to a topic: `/test/message`
* Pulsar en `START LISTENING`

A continuación, se envía un mensaje usando el comando `mosquitto_pub` y, si todo funciona correctamente, se debería recibir el mensaje en Home Assistant:

![Test de integración en HA][3]

> **Nota**: También se puede probar el envío desde el propio HA mediante la opción `Publish a packet`, indicando el _topic_ y el _payload_ (mensaje) y pulsando en `PUBLISH`.

# Actualizar

Si ya se había instalado Mosquitto, se puede actualizar de la siguiente manera:

```
docker stop mosquitto
docker rm mosquitto
docker rmi eclipse-mosquitto
docker-compose up -d
```

# Soporte

Algunos comandos para gestionar la configuración del contenedor:

```
# Acceder al shell mientras el contenedor está ejecutándose
docker exec -it mosquitto /bin/sh

# Monitorizar los logs del contenedor en tiempo real
docker exec -it mosquitto tail -f /mosquitto/log/mosquitto.log
```

# Referencias

* [What is MQTT and How It Works](https://randomnerdtutorials.com/what-is-mqtt-and-how-it-works/) @ Random Nerd Tutorials
* [Install Mosquitto MQTT Broker on Raspberry Pi](https://randomnerdtutorials.com/how-to-install-mosquitto-broker-on-raspberry-pi/) @ Random Nerd Tutorials
* [Testing Mosquitto Broker and Client on Raspberry Pi](https://randomnerdtutorials.com/testing-mosquitto-broker-and-client-on-raspbbery-pi/) @ Random Nerd Tutorials
* [Deploying Mosquitto MQTT broker on Linux using Docker](https://itnext.io/deploying-mosquitto-mqtt-broker-on-linux-using-docker-a9a7fa3a7404)

* [Home Assistant y Node-RED: Configuración rápida](https://www.youtube.com/watch?v=sIvzWJ4ytok) @ Un loco y su tecnología
* [Instalar Home Assistant y Node-RED con Docker](https://youtu.be/wi2b5ZcySuc) @ Un loco y su tecnología
* [Instalar MQTT con Docker utilizando Mosquitto. Integración con Home Assistant y Node Red](https://youtu.be/-Aavz3RKLMc) @ Un loco y su tecnología

# Herramientas

* [MQTT Explorer](https://mqtt-explorer.com/), cliente que proporciona una visión gráfica estructurada de los temas (_topics_) de MQTT

![MQTT Explorer][2]

[1]: /assets/img/blog/2023-02-26_image_1.png "MQTT Broker (c) Cedalo"
[2]: /assets/img/blog/2023-02-26_image_2.png "MQTT Explorer"
[3]: /assets/img/blog/2023-02-26_image_3.png "Test de integración en HA"