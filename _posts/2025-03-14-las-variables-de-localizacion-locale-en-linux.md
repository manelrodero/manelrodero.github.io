---
layout : post
blog-width: true
title: 'Las variables de localización (locale) en Linux'
date: '2025-03-14 16:09:06'
#last-updated: '2025-03-14 16:09:06'
published: true
tags:
- Linux
author:
  display_name: Manel Rodero
#cover-img: "/assets/img/blog/2025-03-14_cover.png"
thumbnail-img: ""
---

En un sistema Linux, las **variables de localización** (`locale`) determinan cómo se deben manejar ciertos aspectos relacionados con la configuración regional, como formatos de idioma, fechas, números, etc.

La configuración estándar y básica que ocurre en varios sistemas Linux, especialmente en escenarios iniciales o predeterminados es la siguiente:

```plaintext
LANG=en_US.UTF-8
LANGUAGE=en_US.UTF-8
LC_CTYPE="C"
LC_NUMERIC="C"
LC_TIME="C"
LC_COLLATE="C"
LC_MONETARY="C"
LC_MESSAGES="C"
LC_PAPER="C"
LC_NAME="C"
LC_ADDRESS="C"
LC_TELEPHONE="C"
LC_MEASUREMENT="C"
LC_IDENTIFICATION="C"
LC_ALL=C
```

Esta configuración es común en servidores o sistemas que necesitan ser lo más genéricos posible, o cuando no se han establecido configuraciones regionales específicas:

- **`LANG=en_US.UTF-8` y `LANGUAGE=en_US.UTF-8`**: Indican que, por defecto, el idioma principal del sistema es el inglés estadounidense y que se utiliza la codificación de caracteres UTF-8, que permite manejar caracteres especiales.

- **`LC_*="C"`**: Este valor indica que cada una de estas categorías usa el entorno "C" o POSIX. Esto significa que no aplica ninguna localización o configuración regional específica; todo sigue un estándar básico y neutral. Es útil para maximizar la compatibilidad o para tareas en las que se necesite precisión y predictibilidad (como ejecutar scripts).

- **`LC_ALL=C`**: La configuración global sobrescribe todas las demás variables `LC_*` para usar el entorno "C". Esto refuerza que no se utilice ninguna localización específica. Es útil para entornos o aplicaciones en los que se necesita evitar el comportamiento dependiente de la región.

# Variables de localización

A continuación, explico para qué sirve cada una de las variables de localización existentes en un sistema Linux:

- **`LANG`**: Configura el idioma general y los formatos regionales por defecto para el sistema. Si no se definen otras variables específicas (como `LC_*`), esta será la usada.
- **`LANGUAGE`**: Específicamente utilizada para la traducción de mensajes y textos del sistema. Puede contener una lista priorizada de idiomas (p.ej., `en_US:es_ES`).
- **`LC_CTYPE`**: Controla cómo se manejan las clasificaciones de caracteres y los límites entre mayúsculas/minúsculas. Por ejemplo, la codificación y el uso de caracteres UTF-8.
- **`LC_NUMERIC`**: Determina el formato de números no monetarios, como el separador decimal (coma o punto) y el separador de miles.
- **`LC_TIME`**: Define cómo se formatean las fechas y las horas (formato de 24h o AM/PM, nombres de meses/días, etc.).
- **`LC_COLLATE`**: Controla cómo se ordenan y comparan cadenas de texto (orden alfabético, caso sensible/insensible).
- **`LC_MONETARY`**: Configura los formatos monetarios, como el símbolo de moneda, separadores y ubicación del símbolo.
- **`LC_MESSAGES`**: Define el idioma en que los programas muestran sus mensajes al usuario, incluyendo errores y advertencias.
- **`LC_PAPER`**: Establece el tamaño estándar del papel (por ejemplo, A4 en Europa o Letter en EE.UU.).
- **`LC_NAME`**: Indica el formato de los nombres de las personas (orden, títulos, etc.).
- **`LC_ADDRESS`**: Configura el formato de direcciones postales en esa región.
- **`LC_TELEPHONE`**: Define el formato estándar de números telefónicos.
- **`LC_MEASUREMENT`**: Establece el sistema de medidas (métrico, imperial, etc.).
- **`LC_IDENTIFICATION`**: Información general sobre la configuración regional, normalmente no usada directamente por los programas.
- **`LC_ALL`**: Si se define, sobrescribe todas las demás variables anteriores con un mismo valor.
- **`LOCPATH`**: Define el directorio en el que el sistema buscará las configuraciones locales instaladas. Por defecto, suele ser `/usr/lib/locale`.
- **`NLSPATH`**: Especifica las rutas de búsqueda para los archivos de mensajes localizados (generalmente relacionados con `LC_MESSAGES`).

# Comandos

Se pueden cambiar estas variables de entorno utilizando el comando `export` en la terminal de Linux:

```bash
# Cambiar una variable específica
export LANG=es_ES.UTF-8

# Cambiar varias variables al mismo tiempo
export LANG=es_ES.UTF-8 LANGUAGE=en_US.UTF-8 LC_ALL=es_ES.UTF-8
```

Después de cambiar las variables, se puede verificar la configuración actual utilizando el comando `locale`.

Si se quiere que las configuraciones sean permanentes, se puede agregar las líneas correspondientes al archivo de configuración de la shell (como `~/.bashrc` o `~/.zshrc`):

```bash
# Hacer los cambios permanentes
echo 'export LANG=es_ES.UTF-8' >> ~/.bashrc
echo 'export LANGUAGE=en_US.UTF-8' >> ~/.bashrc
echo 'export LC_ALL=es_ES.UTF-8' >> ~/.bashrc

# Recargar el archivo de configuración
source ~/.bashrc
```

Si se desea que los cambios se apliquen para todos los usuarios del sistema, se edita el archivo `/etc/default/locale` (requiere permisos de administrador) para añadir o ajustar las líneas correspondientes:

```bash
LANG=en_US.UTF-8
LANGUAGE=en_US.UTF-8
LC_ALL=C
LC_CTYPE=C
```

Al guardar los cambios, todas las sesiones y usuarios tomarán estas configuraciones por defecto.

# Paquetes

Para manejar locales en sistemas basados en Debian (como Debian 12 o Ubuntu), se necesita asegurar que ciertos paquetes estén instalados:

```bash
apt update
apt install locales
```

Este es el paquete básico que proporciona las herramientas necesarias para gestionar los locales en el sistema, como `locale-gen` y `dpkg-reconfigure locales`.

{: .box-note}
Al usar `dkpg-reconfigure locales` se añade `es_ES.UTF-8 UTF-8` y se deja por defecto `en_US.UTF-8`.

Para ver qué locales están disponibles en el sistema, se usa el comando `locale -a`. Esto mostrará una lista de los locales que ya están generados y disponibles para usar:

```plaintext
C
C.utf8
en_US.utf8
es_ES.utf8
POSIX
```

# Soporte de acentos en el editor `nano`

Para que el editor `nano` soporte la escritura de acentos (p.ej. **explicación**) y que éstos sean visibles, se puede utilizar la siguiente configuración:

```
LANG=en_US.UTF-8
LANGUAGE=en_US.UTF-8
#LC_ALL=C
LC_CTYPE=es_ES.UTF-8
```

Al hacerlo, la configuración del sistema queda configurada en:

```plaintext
LANG=en_US.UTF-8
LANGUAGE=en_US.UTF-8
LC_CTYPE=es_ES.UTF-8
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="en_US.UTF-8"
LC_IDENTIFICATION="en_US.UTF-8"
LC_ALL=
```

Se puede observar la diferencia al usar el editor `nano`:

![Editor nano con es-ES.UTF-8][1]

### Historial de cambios

* **2025-03-14**: Documento inicial

[1]: /assets/img/blog/2025-03-14_image_1.png "Editor nano con es-ES.UTF-8"
