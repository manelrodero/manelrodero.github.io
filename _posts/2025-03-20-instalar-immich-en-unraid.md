---
layout : post
blog-width: true
title: 'Instalar Immich en Unraid'
date: '2025-03-20 19:06:31'
last-updated: '2025-09-21 17:21:31'
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

A fecha de hoy, 20 de septiembre de 2025, el fichero `docker-compose.yml` es el siguiente:

```yaml
#
# WARNING: To install Immich, follow our guide: https://immich.app/docs/install/docker-compose
#
# Make sure to use the docker-compose.yml of the current release:
#
# https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
#
# The compose file on main may not be compatible with the latest release.

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
      - ${UPLOAD_LOCATION}:/data
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
    # For hardware acceleration, add one of -[armnn, cuda, rocm, openvino, rknn] to the image tag.
    # Example tag: ${IMMICH_VERSION:-release}-cuda
    image: ghcr.io/immich-app/immich-machine-learning:${IMMICH_VERSION:-release}
    # extends: # uncomment this section for hardware acceleration - see https://immich.app/docs/features/ml-hardware-acceleration
    #   file: hwaccel.ml.yml
    #   service: cpu # set to one of [armnn, cuda, rocm, openvino, openvino-wsl, rknn] for accelerated inference - use the `-wsl` version for WSL2 where applicable
    volumes:
      - model-cache:/cache
    env_file:
      - .env
    restart: always
    healthcheck:
      disable: false

  redis:
    container_name: immich_redis
    image: docker.io/valkey/valkey:8-bookworm@sha256:fea8b3e67b15729d4bb70589eb03367bab9ad1ee89c876f54327fc7c6e618571
    healthcheck:
      test: redis-cli ping || exit 1
    restart: always

  database:
    container_name: immich_postgres
    image: ghcr.io/immich-app/postgres:14-vectorchord0.4.3-pgvectors0.2.0@sha256:8d292bdb796aa58bbbaa47fe971c8516f6f57d6a47e7172e62754feb6ed4e7b0
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_DB: ${DB_DATABASE_NAME}
      POSTGRES_INITDB_ARGS: '--data-checksums'
      # Uncomment the DB_STORAGE_TYPE: 'HDD' var if your database isn't stored on SSDs
      # DB_STORAGE_TYPE: 'HDD'
    volumes:
      # Do not edit the next line. If you want to change the database storage location on your system, edit the value of DB_DATA_LOCATION in the .env file
      - ${DB_DATA_LOCATION}:/var/lib/postgresql/data
    shm_size: 128mb
    restart: always

volumes:
  model-cache:
```

# Unraid

Unraid proporciona una interfaz gráfica fácil de usar para configurar, instalar y administrar contenedores Docker. Para ello utiliza el complemento **Community Applications** que permite buscar y añadir contenedores con solo unos clics.

Este complemento incluye una biblioteca enorme de archivos preconfigurados que contienen toda la información necesaria para ejecutar el contenedor Docker de una aplicación en Unraid.

Estos archivos preconfigurados son **plantillas** que incluyen configuraciones específicas, como las rutas de almacenamiento, puertos, variables de entorno y otros parámetros necesarios para que el contenedor funcione correctamente:

![Docker en Unraid][1]

En esta biblioteca pueden existir plantillas de diferentes usuarios para la misma aplicación. Esto es debido a que esta tienda de aplicaciones se construye de forma colaborativa por los usuarios de la comunidad Unraid.

Cada plantilla tiene una descripción donde se detallan los requerimientos necesarios para usarla, un botón para instalarla y enlaces a la página del proyecto, el registro de la imagen Docker y el _post_ de soporte en el foro de Unraid:

![Community Applications][2]

## Immich desde Community Apps

Para ejecutar Immich en Unraid, la mayoría de usuarios suele seguir las recomendaciones de Edward Rawlings en su canal [Spaceinvader One](https://www.youtube.com/@SpaceinvaderOne){:target="_blank"}.

Este [usuario avanzado de Unraid](https://forums.unraid.net/profile/67288-spaceinvaderone/){:target="_blank"} publicó hace un año estos tres vídeos sobre la instalación de Immich en un ordenador con un procesador Intel N100 para demostrar la eficiencia del programa y los pocos recursos que necesita:

* [Immich Pt 1 - A Closer Look - See How It Compares with Google Photos Before You Switch](https://www.youtube.com/watch?v=krAZcUIP3YE){:target="_blank"}
* [Immich Pt 2 - Easy Setup - Install, Connect, and Upload in Minutes!](https://www.youtube.com/watch?v=LtNWxxM5Mzg){:target="_blank"}
* [Immich Pt 3 - Your Photos, Your Rules - Google Takeout to Immich Transfer](https://www.youtube.com/watch?v=tCylg2AHA8c){:target="_blank"}

{: .box-note}
El único **vídeo específico sobre Unraid** es el segundo, donde se explica su instalación usando las plantillas de Community Apps. El primero es una comparativa bastante interesante de Immich con Google Photos y el tercero explica cómo usar Google Takeout para descargar las fotos e importarlas después en Immich.

Para replicar su configuración habría que instalar un par de plantillas desde las Community Apps:

* [PostgreSQL_Immich](https://github.com/SpaceinvaderOne/Docker-Templates-Unraid/blob/master/spaceinvaderone/PostgreSQL_Immich.xml){:target="_blank"} (SpaceInvaderOne)
* [Immich](https://github.com/imagegenius/templates/blob/main/unraid/immich.xml){:target="_blank"} (imagegenius)

La primera plantilla es un contenedor basado en [**PostgreSQL 16**](https://www.postgresql.org/){:target="_blank"} específicamente configurado para una integración fluida con el contenedor de Immich. Immich requiere PostgreSQL equipado con [**pgvecto.rs**](https://github.com/tensorchord/pgvecto.rs){:target="_blank"}, una extensión que habilita funciones de búsqueda de similitud vectorial.

La versión indicada en la plantilla es `tensorchord/pgvecto-rs:pg16-v0.3.0` pero se podría utilizar la versión de la imagen que se incluye en el fichero [`docker-compose.yml` de Immich](https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml), `immich-app/postgres:14-vectorchord0.4.3-pgvectors0.2.0`.

{: .box-note}
Immich reemplazó el proyecto pgvecto.rs por [VectorChord](https://github.com/tensorchord/VectorChord/){:target="_blank"}, una versión más rápida y escalable, en [mayo de 2025](https://github.com/immich-app/immich/discussions/18429){:target="_blank"}.

La segunda plantilla es el contenedor de **Immich** propiamente dicho preparado para ejecutar [**Redis**](https://redis.io/){:target="_blank"}, una base de datos de código abierto, en memoria, que funciona como un almacén de estructuras de datos clave-valor.

Se puede usar un _docker mod_ para ejecutar Redis dentro de este contenedor añadiendo las variables `DOCKER_MODS=imagegenius/mods:universal-redis` y `REDIS_HOSTNAME=localhost`.

## Immich con plantillas propias

Aunque el uso de las _Community Apps Templates_ permite instalar y ejecutar Immich de forma rápida, [**la imagen `ghcr.io/imagegenius/immich:release` utilizada no está soportada oficialmente**](https://immich.app/docs/install/unraid){:target="_blank"}.

Por este motivo, he decidido crear mis propias **plantillas** usando **ficheros XML** para poder crear contenedores Docker a partir de ellas.

El contenido del fichero `my-immich-postgresql.xml` es el siguiente:

```xml
<?xml version="1.0"?>
<Container version="2">
    <Name>immich-postgresql</Name>
    <Repository>ghcr.io/immich-app/postgres:14-vectorchord0.4.3-pgvectors0.2.0</Repository>
    <Network>immich-net</Network>
    <Shell>bash</Shell>
    <Privileged>false</Privileged>
    <Icon>https://raw.githubusercontent.com/homarr-labs/dashboard-icons/refs/heads/main/png/postgres.png</Icon>
    <ExtraParams>--hostname database --dns=1.1.1.1 --shm-size=128m</ExtraParams>
    <Config Name="POSTGRES_PASSWORD" Target="POSTGRES_PASSWORD" Default="postgres" Mode="" Description="Initial superuser password (required). Set once at first database creation." Type="Variable" Display="always" Required="true" Mask="true">postgres</Config>
    <Config Name="POSTGRES_USER" Target="POSTGRES_USER" Default="postgres" Mode="" Description="Initial superuser name (default: postgres)" Type="Variable" Display="always" Required="true" Mask="false">postgres</Config>
    <Config Name="POSTGRES_DB" Target="POSTGRES_DB" Default="immich" Mode="" Description="Initial database name (default: immich)" Type="Variable" Display="always" Required="true" Mask="false">immich</Config>
    <Config Name="POSTGRES_INITDB_ARGS" Target="POSTGRES_INITDB_ARGS" Default="--data-checksums" Mode="" Description="Database initialization parameters" Type="Variable" Display="advanced" Required="false" Mask="false">--data-checksums</Config>
    <Config Name="DB_STORAGE_TYPE" Target="DB_STORAGE_TYPE" Default="" Mode="" Description="Set to 'HDD' if database is stored on spinning disks" Type="Variable" Display="advanced" Required="false" Mask="false"></Config>
    <Config Name="Database storage path (appdata)" Target="/var/lib/postgresql/data" Default="/mnt/user/appdata/postgresql_immich" Mode="rw" Description="PostgreSQL data storage location" Type="Path" Display="always" Required="true" Mask="false">/mnt/user/appdata/immich-postgresql</Config>
    <Config Name="PostgreSQL access port" Target="5432" Default="5432" Mode="tcp" Description="PostgreSQL TCP connection port mapped to Docker Bridge NAT" Type="Port" Display="always" Required="true" Mask="false">5432</Config>
    <Config Name="TZ" Target="TZ" Default="Europe/Madrid" Mode="" Description="Container time zone" Type="Variable" Display="always" Required="false" Mask="false">Europe/Madrid</Config>
</Container>
```

El contenido del fichero `my-immich-valkey.xml` es el siguiente:

```xml
<?xml version="1.0"?>
<Container version="2">
    <Name>immich-valkey</Name>
    <Repository>docker.io/valkey/valkey:8-bookworm</Repository>
    <Network>immich-net</Network>
    <Shell>bash</Shell>
    <Privileged>false</Privileged>
    <Icon>https://raw.githubusercontent.com/homarr-labs/dashboard-icons/refs/heads/main/png/valkey.png</Icon>
    <ExtraParams>--hostname redis --dns=1.1.1.1</ExtraParams>
    <Config Name="Data storage path (appdata)" Target="/data" Default="/mnt/user/appdata/immich-redis" Mode="rw" Description="Valkey (Redis) data storage location" Type="Path" Display="advanced" Required="false" Mask="false">/mnt/user/appdata/immich-valkey</Config>
    <Config Name="Valkey (Redis) access port" Target="6379" Default="6379" Mode="tcp" Description="Valkey (Redis) TCP connection port mapped to Docker Bridge NAT" Type="Port" Display="always" Required="true" Mask="false">6379</Config>
    <Config Name="TZ" Target="TZ" Default="Europe/Madrid" Mode="" Description="Container time zone" Type="Variable" Display="always" Required="false" Mask="false">Europe/Madrid</Config>
</Container>
```

El contenido del fichero `my-immich-machine-learning.xml` es el siguiente:

```xml
<?xml version="1.0"?>
<Container version="2">
    <Name>immich-machine-learning</Name>
    <Repository>ghcr.io/immich-app/immich-machine-learning:v1.142.1</Repository>
    <Network>immich-net</Network>
    <Shell>bash</Shell>
    <Privileged>false</Privileged>
    <Icon>https://raw.githubusercontent.com/homarr-labs/dashboard-icons/refs/heads/main/png/immich.png</Icon>
    <ExtraParams>--hostname immich-ml --dns=1.1.1.1</ExtraParams>
    <Config Name="IMMICH_LOG_LEVEL" Target="IMMICH_LOG_LEVEL" Default="log" Mode="" Description="Log level (verbose, debug, log, warn, error)" Type="Variable" Display="advanced" Required="false" Mask="false">log</Config>
    <Config Name="Machine Learning model cache" Target="/cache" Default="/mnt/user/appdata/immich-machine-learning/cache" Mode="rw" Description="Directory where models are downloaded" Type="Path" Display="advanced" Required="true" Mask="false">/mnt/user/appdata/immich-machine-learning/cache</Config>
    <Config Name="TZ" Target="TZ" Default="Europe/Madrid" Mode="" Description="Container time zone" Type="Variable" Display="always" Required="false" Mask="false">Europe/Madrid</Config>
</Container>
```

El contenido del fichero `my-immich-server` es el siguiente:

```xml
<?xml version="1.0"?>
<Container version="2">
    <Name>immich-server</Name>
    <Repository>ghcr.io/immich-app/immich-server:v1.142.1</Repository>
    <Network>immich-net</Network>
    <Shell>bash</Shell>
    <Privileged>false</Privileged>
    <WebUI>http://[IP]:[PORT:2283]</WebUI>
    <TemplateURL/>
    <Icon>https://raw.githubusercontent.com/homarr-labs/dashboard-icons/refs/heads/main/png/immich.png</Icon>
    <ExtraParams>--hostname immich</ExtraParams>
    <Config Name="DB_USERNAME" Target="DB_USERNAME" Default="postgres" Mode="" Description="PostgreSQL Username" Type="Variable" Display="always" Required="true" Mask="false">postgres</Config>
    <Config Name="DB_PASSWORD" Target="DB_PASSWORD" Default="postgres" Mode="" Description="PostgreSQL Password" Type="Variable" Display="always" Required="true" Mask="true">postgres</Config>
    <Config Name="DB_DATABASE_NAME" Target="DB_DATABASE_NAME" Default="immich" Mode="" Description="PostgreSQL Database Name" Type="Variable" Display="always" Required="true" Mask="false">immich</Config>
    <Config Name="IMMICH_LOG_LEVEL" Target="IMMICH_LOG_LEVEL" Default="log" Mode="" Description="Log level (verbose, debug, log, warn, error)" Type="Variable" Display="advanced" Required="false" Mask="false">log</Config>
    <Config Name="Upload Location" Target="/data" Default="/mnt/user/photos/immich" Mode="rw" Description="Directory where uploaded photos and videos are stored" Type="Path" Display="always" Required="true" Mask="false">/mnt/user/photos/immich</Config>
    <Config Name="Immich Web Port" Target="2283" Default="2283" Mode="tcp" Description="Web interface port for Immich" Type="Port" Display="always" Required="true" Mask="false">2283</Config>
    <Config Name="Time Zone" Target="TZ" Default="Europe/Madrid" Mode="" Description="Container time zone" Type="Variable" Display="always" Required="false" Mask="false">Europe/Madrid</Config>
</Container>
```

Estos ficheros se copian al directorio `/boot/config/plugins/dockerMan/templates-user` para que se puedan utilizar a la hora de crear un contenedor Docker:

```plaintext
root@unraid:/boot/config/plugins/dockerMan/templates-user# ls -l
total 64
-rw------- 1 root root 2229 Sep 21 16:55 my-immich-machine-learning.xml
-rw------- 1 root root 3120 Sep 21 13:26 my-immich-postgresql.xml
-rw------- 1 root root 3184 Sep 21 16:21 my-immich-server.xml
-rw------- 1 root root 1890 Sep 21 13:24 my-immich-valkey.xml
```

Para que los contenedores funcionen correctamente y se puedan comunicar entre ellos usando sus nombres de _host_ es necesario crear una **red** de tipo Bridge llamada **immich-net**:

```bash
docker network create immich-net
```

A continuación se puede acceder a la sección `Docker`, pulsar el botón `Add Container` y seleccionar la plantilla al final de la lista para crear cada uno de los contenedores.

Los contenedores se crearán en el orden mostrado en la siguiente imagen para que cada servicio esté disponible cuando se requiere:

![Immich en Unraid][9]

Si todo funciona correctamente, se podrá acceder a la interfaz web de Immich pulsando en el Docker correspondiente y seleccionando la opción `WebUI`.

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
* **2025-09-21**: Plantillas personalizadas

[1]: /assets/img/blog/2025-03-20_image_1.png "Docker en Unraid"
[2]: /assets/img/blog/2025-03-20_image_2.png "Community Applications"
[3]: /assets/img/blog/2025-03-20_image_3.png "Immich Interface"
[4]: /assets/img/blog/2025-03-20_image_4.png "First Photos"
[5]: /assets/img/blog/2025-03-20_image_5.png "Metadatos y Mapas"
[6]: /assets/img/blog/2025-03-20_image_6.png "Fotos de personas"
[7]: /assets/img/blog/2025-03-20_image_7.png "Reconocimiento"
[8]: /assets/img/blog/2025-03-20_image_8.png "Aplicación Android"
[9]: /assets/img/blog/2025-03-20_image_9.png "Immich en Unraid"