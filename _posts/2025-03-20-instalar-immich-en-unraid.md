---
layout : post
blog-width: true
title: 'Instalar Immich en Unraid'
date: '2025-03-20 19:06:31'
last-updated: '2025-03-24 22:06:31'
published: true
tags:
- Unraid
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2025-03-20_cover.png"
thumbnail-img: ""
---

[**Immich**](https://immich.app/){:target="_blank"} es una aplicación de código abierto diseñada para la gestión y almacenamiento de fotos y videos personales. Su propósito principal es ofrecer una alternativa privada y autosuficiente a servicios como Google Photos o iCloud, permitiendo a los usuarios mantener el control total sobre sus datos.

Sus características principales son las siguientes:

* **Almacenamiento privado**: Todas las fotos y videos se alojan en nuestro propio servidor, garantizando que los datos permanezcan bajo nuestro control.
* **Organización automática**: Utiliza metadatos como fechas, ubicaciones y carpetas para organizar los archivos de manera eficiente.
* **Respaldo automático**: Se puede configurar la sincronización automática desde dispositivos móviles para que nuestras fotos y videos se respalden automáticamente.
* **Búsqueda avanzada**: Ofrece funciones de búsqueda basadas en texto, etiquetas y fechas.
* **Uso compartido**: Permite compartir fotos o álbumes con otros usuarios, ideal para familias o grupos.

# Docker Compose

Antes de comenzar la instalación en Unraid, es recomendable revisar la documentación oficial para [instalar Immich usando Docker Compose](https://immich.app/docs/install/docker-compose){:target="_blank"} que consiste en:

* Crear un directorio para el contenedor

```Bash
mkdir ./immich-app
cd ./immich-app
```

* Descargar los ficheros `docker-compose.yml` y `.env`

```Bash
wget -O docker-compose.yml https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
wget -O .env https://github.com/immich-app/immich/releases/latest/download/example.env
```

* Editar el fichero `.env` para asignar valores a las variables según nuestro entorno

```Plaintext
# Ubicación donde se almacenarán las fotos que subamos
UPLOAD_LOCATION=./library

# Ubicación de los ficheros de la base de datos Postgres
DB_DATA_LOCATION=./postgres

# Zona horaria (ver referencias)
TZ=Paris/Madrid

# Versión de Immich que queremos usar (release, v1.129.0, etc.)
IMMICH_VERSION=release

# Password para la autenticación local de Postgres (16 caracteres, A-Za-z0-9)
DB_PASSWORD=R7m9W3n2Y6p8K4v1

DB_USERNAME=postgres
DB_DATABASE_NAME=immich
```

* Poner en marcha los contenedores

```Bash
docker compose up -d
```

A día de hoy, el fichero `docker-compose.yml` es el siguiente:

```yaml
name: immich
services:
  immich-server:
    container_name: immich_server
    image: ghcr.io/immich-app/immich-server:${IMMICH_VERSION:-release}
    # extends:
    #   file: hwaccel.transcoding.yml
    #   service: cpu # set to one of [nvenc, quicksync, rkmpp, vaapi, vaapi-wsl] for accelerated transcoding
    volumes:
      # Do not edit the next line. If you want to change the media storage location on your system, edit the value of UPLOAD_LOCATION in the .env file
      - ${UPLOAD_LOCATION}:/usr/src/app/upload
      - /etc/localtime:/etc/localtime:ro
    env_file:
      - .env
    ports:
      - '2283:2283'
    depends_on:
      - redis
      - database
    restart: always
    healthcheck:
      disable: false

  immich-machine-learning:
    container_name: immich_machine_learning
    # For hardware acceleration, add one of -[armnn, cuda, openvino] to the image tag.
    # Example tag: ${IMMICH_VERSION:-release}-cuda
    image: ghcr.io/immich-app/immich-machine-learning:${IMMICH_VERSION:-release}
    # extends: # uncomment this section for hardware acceleration - see https://immich.app/docs/features/ml-hardware-acceleration
    #   file: hwaccel.ml.yml
    #   service: cpu # set to one of [armnn, cuda, openvino, openvino-wsl] for accelerated inference - use the `-wsl` version for WSL2 where applicable
    volumes:
      - model-cache:/cache
    env_file:
      - .env
    restart: always
    healthcheck:
      disable: false

  redis:
    container_name: immich_redis
    image: docker.io/redis:6.2-alpine@sha256:148bb5411c184abd288d9aaed139c98123eeb8824c5d3fce03cf721db58066d8
    healthcheck:
      test: redis-cli ping || exit 1
    restart: always

  database:
    container_name: immich_postgres
    image: docker.io/tensorchord/pgvecto-rs:pg14-v0.2.0@sha256:739cdd626151ff1f796dc95a6591b55a714f341c737e27f045019ceabf8e8c52
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_DB: ${DB_DATABASE_NAME}
      POSTGRES_INITDB_ARGS: '--data-checksums'
    volumes:
      # Do not edit the next line. If you want to change the database storage location on your system, edit the value of DB_DATA_LOCATION in the .env file
      - ${DB_DATA_LOCATION}:/var/lib/postgresql/data
    healthcheck:
      test: >-
        pg_isready --dbname="$${POSTGRES_DB}" --username="$${POSTGRES_USER}" || exit 1;
        Chksum="$$(psql --dbname="$${POSTGRES_DB}" --username="$${POSTGRES_USER}" --tuples-only --no-align
        --command='SELECT COALESCE(SUM(checksum_failures), 0) FROM pg_stat_database')";
        echo "checksum failure count is $$Chksum";
        [ "$$Chksum" = '0' ] || exit 1
      interval: 5m
      start_interval: 30s
      start_period: 5m
    command: >-
      postgres
      -c shared_preload_libraries=vectors.so
      -c 'search_path="$$user", public, vectors'
      -c logging_collector=on
      -c max_wal_size=2GB
      -c shared_buffers=512MB
      -c wal_compression=on
    restart: always

volumes:
  model-cache:
```

# Docker en Unraid

Unraid proporciona una interfaz gráfica fácil de usar para configurar, instalar y administrar contenedores Docker. Para ello utiliza el complemento **Community Applications** que permite buscar y añadir contenedores con solo unos clics.

Este complemento incluye una biblioteca enorme de archivos preconfigurados que contienen toda la información necesaria para ejecutar el contenedor Docker de una aplicación en Unraid.

Estos archivos preconfigurados son **plantillas** que incluyen configuraciones específicas, como las rutas de almacenamiento, puertos, variables de entorno y otros parámetros necesarios para que el contenedor funcione correctamente.

![Docker en Unraid][1]

En esta biblioteca pueden existir plantillas de diferentes usuarios para la misma aplicación. Esto es debido a que esta tienda de aplicaciones se construye de forma colaborativa por los usuarios de la comunidad Unraid.

Cada plantilla tiene una descripción donde se detallan los requerimientos necesarios para usarla, un botón para instalarla y enlaces a la página del proyecto, el registro de la imagen Docker y el _post_ de soporte en el foro de Unraid.

![Community Applications][2]

## Immich

Para ejecutar Immich en Unraid, la mayoría de usuarios suele seguir las recomendaciones de Edward Rawlings en su canal [Spaceinvader One](https://www.youtube.com/@SpaceinvaderOne){:target="_blank"}.

Este [usuario avanzado de Unraid](https://forums.unraid.net/profile/67288-spaceinvaderone/){:target="_blank"} publicó hace un año estos tres vídeos sobre la instalación de Immich en un ordenador con un procesador Intel N100 para demostrar la eficiencia del programa y los pocos recursos que necesita:

* [Immich Pt 1 - A Closer Look - See How It Compares with Google Photos Before You Switch](https://www.youtube.com/watch?v=krAZcUIP3YE){:target="_blank"}
* [Immich Pt 2 - Easy Setup - Install, Connect, and Upload in Minutes!](https://www.youtube.com/watch?v=LtNWxxM5Mzg){:target="_blank"}
* [Immich Pt 3 - Your Photos, Your Rules - Google Takeout to Immich Transfer](https://www.youtube.com/watch?v=tCylg2AHA8c){:target="_blank"}

{: .box-note}
El único **vídeo específico sobre Unraid** es el segundo, donde se explica su instalación usando las plantillas de Community Apps. El primero es una comparativa bastante interesante de Immich con Google Photos y el tercero explica cómo usar Google Takeout para descargar las fotos e importarlas después en Immich.

Para replicar su configuración hay que instalar un par de plantillas desde las Community Apps:

* [PostgreSQL_Immich](https://github.com/SpaceinvaderOne/Docker-Templates-Unraid/blob/master/spaceinvaderone/PostgreSQL_Immich.xml){:target="_blank"} (SpaceInvaderOne)
* [Immich](https://github.com/imagegenius/templates/blob/main/unraid/immich.xml){:target="_blank"} (imagegenius)

La primera es un contenedor basado en [**PostgreSQL 16**](https://www.postgresql.org/){:target="_blank"} específicamente configurado para una integración fluida con el contenedor de Immich. Immich requiere PostgreSQL equipado con [**pgvecto.rs**](https://github.com/tensorchord/pgvecto.rs){:target="_blank"}, una extensión que habilita funciones de búsqueda de similitud vectorial.

La versión indicada en la plantilla es `tensorchord/pgvecto-rs:pg16-v0.3.0` pero se podría utilizar la versión de la imagen que se incluye en el fichero [`docker-compose.yml` de Immich](https://github.com/immich-app/immich/blob/main/docker/docker-compose.yml), `tensorchord/pgvecto-rs:pg14-v0.2.0`.

{: .box-note}
Este proyecto está siendo reemplazado por [VectorChord](https://github.com/tensorchord/VectorChord/){:target="_blank"}, una versión más rápida y escalable, pero Immich aún no la ha adoptado.

Al aplicar los parámetros de la plantilla, Unraid ejecuta el siguiente comando `docker run`:

```plaintext
docker run
  -d
  --name='PostgreSQL_Immich'
  --net='bridge'
  --pids-limit 2048
  -e TZ="Europe/Paris"
  -e HOST_OS="Unraid"
  -e HOST_HOSTNAME="unraid"
  -e HOST_CONTAINERNAME="PostgreSQL_Immich"
  -e 'POSTGRES_PASSWORD'='R7m9W3n2Y6p8K4v1'
  -e 'POSTGRES_USER'='postgres'
  -e 'POSTGRES_DB'='immich'
  -l net.unraid.docker.managed=dockerman
  -l net.unraid.docker.icon='https://github.com/SpaceinvaderOne/Docker-Templates-Unraid/blob/master/spaceinvaderone/docker_icons/postgresql-immich-logo.png?raw=true'
  -p '5433:5432/tcp'
  -v '/mnt/user/appdata/PostgreSQL_Immich':'/var/lib/postgresql/data':'rw' 'tensorchord/pgvecto-rs:pg16-v0.3.0'
```

La segunda plantilla se corresponde al contenedor de **Immich** propiamente dicho preparado para ejecutar [**Redis**](https://redis.io/){:target="_blank"}, una base de datos de código abierto, en memoria, que funciona como un almacén de estructuras de datos clave-valor.

Se puede usar un _docker mod_ para ejecutar Redis dentro de este contenedor añadiendo las variables `DOCKER_MODS=imagegenius/mods:universal-redis` y `REDIS_HOSTNAME=localhost`.

Al aplicar los parámetros de la plantilla, Unraid ejecuta el siguiente comando `docker run`:

```plaintext
docker run
  -d
  --name='immich'
  --net='bridge'
  --pids-limit 2048
  -e TZ="Europe/Paris"
  -e HOST_OS="Unraid"
  -e HOST_HOSTNAME="unraid"
  -e HOST_CONTAINERNAME="immich"
  -e 'DB_HOSTNAME'='192.168.1.60'
  -e 'DB_USERNAME'='postgres'
  -e 'DB_PASSWORD'='R7m9W3n2Y6p8K4v1'
  -e 'DB_DATABASE_NAME'='immich'
  -e 'REDIS_HOSTNAME'='localhost'
  -e 'DB_PORT'='5433'
  -e 'REDIS_PORT'='6379'
  -e 'REDIS_PASSWORD'='XlqRGn8vsTKmWhiyfpz5Jt7Lb'
  -e 'MACHINE_LEARNING_HOST'='0.0.0.0'
  -e 'MACHINE_LEARNING_PORT'='3003'
  -e 'MACHINE_LEARNING_WORKERS'='1'
  -e 'MACHINE_LEARNING_WORKER_TIMEOUT'='120'
  -e 'DOCKER_MODS'='imagegenius/mods:universal-redis'
  -e 'PUID'='99'
  -e 'PGID'='100'
  -e 'UMASK'='022'
  -l net.unraid.docker.managed=dockerman
  -l net.unraid.docker.webui='http://[IP]:[PORT:8080]'
  -l net.unraid.docker.icon='https://raw.githubusercontent.com/imagegenius/templates/main/unraid/img/immich.png'
  -p '8080:8080/tcp'
  -v '/mnt/user/photos/Immich/':'/photos':'rw'
  -v '/mnt/user/photos/Library/':'/libraries':'rw'
  -v '/mnt/user/appdata/immich':'/config':'rw' 'ghcr.io/imagegenius/immich:cuda'
```

# Configuración de Immich

La primera vez que se accede a la interfaz web de Immich, hay que pulsar el botón `Getting Started` para registrar los datos del **usuario administrador** (dirección de correo, contraseña y nombre) de Immich.

A continuación se inicia sesión con este usuario y se completa el asistente para indicar:

* El **tema de color** que se utilizará (`Light` o `Dark`)
* La **privacidad** (utilizar servicios externos para **mapas** o la **comprobación de versión** en GitHub)
* Habilitar las **plantillas de almacenamiento** (es interesante si se quiera organizar la librería en `YYYY\YYYY-MM` o similar)

La interfaz principal de Immich es muy similar a Google Photos tal como se puede observar en la siguiente imagen:

![Immich Interface][3]

Los ajustes generales del servidor se configuran pulsando en el icono de nuestro usuario y después el botón **`Administration`**:

* **Users**: creación de usuarios, asignación de cuotas y  asignar **storage label** para organizar las fotos del usuario bajo una carpeta con ese nombre.
* **Jobs**: monitorizar y ejecutar diferentes tareas (generación de _thumbnails_, extracción de metadatos, escaneo de librerías externas, etc.)
* **Settings**: opciones de autenticación, copias de seguridad, calidad de las imágenes, metadatos, librerías externas, [_**machine learning**_](https://immich.app/docs/features/ml-hardware-acceleration/){:target="_blank"}, etc.
* **External Libraries**: definición de [**librerías externas**](https://immich.app/docs/features/libraries){:target="_blank"} que ya tuviésemos antes de Immich
* **Server Stats**: estadísticas del servidor (número de fotos y vídeos totales, por usuario, etc.)

Los ajustes de cada cuenta se configuran pulsando en el icono de nuestro usuario y después el botón **`Account Settings`**:

* **App Settings**: idioma, formato de fecha, _locale_, etc.
* **Account**: correo electrónico, nombre, etc.
* **Account usage statistics**: número de fotos/videos favoritos, archivados, en la papelera, etc.
* **API Keys**: API para realizar operaciones desde CLI
* **Authorized Devices**: Dispositivos conectados a esta cuenta
* **Download**: Tamaño máximo de las descargas
* **Features**: Vistas de carpetas y de personas, _star rating_, _tags_, etc.
* **Notifications**: Notificaciones por correo electrónico
* **Password**: Cambio de contraseña
* **Partner Sharing**: Compartir fotos con otros usuarios
* **Purchase**: Colaboración con el proyecto (100$ por servidor o $25 por usuario, _lifetime purchase_)

## Subir las primeras fotos

Subir fotos a Immich es bastante sencillo, únicamente hay que pulsar el botón `Click to upload your first photos` de la página principal y seleccionar las fotos desde nuestro ordenador:

Por ejemplo, aquí se han subido unas [imágenes de cactus obtenidas de Archive.org](https://archive.org/details/various-nature-images-zip){:target="_blank"} para poder mostrarlas tranquilamente en este artículo.

Como se puede observar, las fotos se ordenan por fecha y en la parte derecha hay un **_timeline_** (una de las características que lo diferencian de PhotoPrism):

![First Photos][4]

Al pulsar en cualquiera de las fotos se pueden ver los metadatos que contiene la fotografía (fecha en que se tomó, cámara fotográfica, localización, etc.). Si las fotografías están geolocalizadas, se podría navegar por un mapa para buscarlas:

![Metadatos y Mapas][5]

Una característica muy interesante es el reconocimiento de caras y agrupación de las fotos de esa persona. Por ejemplo, después de subir unas cuantas de Lamine Yamal y de Leo Messi descargadas desde Google, se ejecutan los trabajos de **Face Detection** y de **Facial Recognition**:

![Fotos de personas][6]

Automáticamente, aparecen los dos personajes en la vista `People` del menú lateral y podemos navegar a las fotos correspondientes a cada uno de ellos:

![Reconocimiento][7]

{: .box-note}
Por defecto, Immich creará una persona si hay un mínimo de 3 fotos en la que ésta aparezca. En este ejemplo, ha habido que bajar este valor a 2 para que aparezca Leo Messi. ¿Sabes por qué? ;-)

A partir de aquí se abren un montón de posibilidades con todas las opciones que tiene este programa. Únicamente es cuestión de dedicarle un rato para explorar cada menú y cada configuración para adaptarla a nuestro entorno o flujo de trabajo.

## Cliente para móvil

Immich tiene cliente para [**Android**](https://play.google.com/store/apps/details?id=app.alextran.immich){:target="_blank"} y para [**iOS**](https://apps.apple.com/us/app/immich/id1613945652){:target="_blank"}. En mi caso he instalado el primero y, después de ejecutarlo por primera vez, he configurado la URL del servidor y he iniciado sesión con mi usuario y mi contraseña.

En la siguiente captura de pantalla se pueden observar las imágenes existentes en el servidor (con el icono de una **nube**) y las imágenes que únicamente están en mi móvil (las 3 fotos de libros con el icono de una **nube tachada**):

![Aplicación Android][8]

Para subirlas, únicamente he de marcarlas y pulsar el botón para enviarlas al servidor. Obviamente, se puede configurar para que esta operación sea automática, para que se realiza en determinadas condiciones de red, etc.

Las aplicaciones móviles también tienen muchas opciones de configuración que hay que explorar. ¡Te toca comenzar a jugar!

# Referencias

* [How to Self Host and Install Immich Photo Server on Unraid](https://www.youtube.com/watch?v=RCmPAIC74Hg){:target="_blank"}, TheDadNerd
* [Como instalar immich en Home Assistant, TrueNAS y Docker](https://www.youtube.com/watch?v=7jJG2wAdcFM){:target="_blank"}, Jonatan Castro
* [Don't Let Apple & Google Harvest Your Photos, Use Immich to Self-Host Your Own Cloud!](https://www.youtube.com/watch?v=URJiQb8PwWo&t=9s){:target="_blank"}, Jim's Garage

### Historial de cambios

* **2025-03-20**: Documento inicial
* **2025-03-24**: Imágenes y configuración

[1]: /assets/img/blog/2025-03-20_image_1.png "Docker en Unraid"
[2]: /assets/img/blog/2025-03-20_image_2.png "Community Applications"
[3]: /assets/img/blog/2025-03-20_image_3.png "Immich Interface"
[4]: /assets/img/blog/2025-03-20_image_4.png "First Photos"
[5]: /assets/img/blog/2025-03-20_image_5.png "Metadatos y Mapas"
[6]: /assets/img/blog/2025-03-20_image_6.png "Fotos de personas"
[7]: /assets/img/blog/2025-03-20_image_7.png "Reconocimiento"
[8]: /assets/img/blog/2025-03-20_image_8.png "Aplicación Android"
