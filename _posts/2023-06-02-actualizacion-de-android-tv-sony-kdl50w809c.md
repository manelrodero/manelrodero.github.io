---
layout : post
blog-width: true
title: 'Actualización de Android TV Sony KDL-50W809C'
date: '2023-06-02 20:21:42'
published: true
tags:
- Android
author:
  display_name: Manel Rodero
---

# Introducción

Hace bastante tiempo que tenemos una televisión [Sony KDL-50W809C](https://www.sony.es/electronics/support/televisions-projectors-lcd-tvs-android-/kdl-50w809c){:target="_blank"} con [Android TV](https://www.android.com/intl/es_es/tv/){:target="_blank"} y últimamente ha comenzado a "hacer el tonto": reinicios inesperados, menús que no responden o aplicaciones que no se abren correctamente.

La versión de _firmware_ que tiene actualmente es la **5.433** y la instalé a mediados del año 2019. Aunque la televisión no indica que haya actualizaciones, en la página web de Sony está disponible la [versión 5.457 del firmware](https://www.sony.es/electronics/support/lcd-tvs-android-w75xc_w80xc-series/kdl-50w809c/software/00259461){:target="_blank"} por lo que he decidido descargarla y actualizar directamente desde una unidad _flash_.

# Actualización

El proceso de actualización ha sido bastante sencillo:

* Descargar el fichero [`sony_tvupdate_2015_5457_eua_auth.zip`](https://tv.update.sony.net/OND/0000/eu/main/PS116288/sony_tvupdate_2015_5457_eua_auth.zip){:target="_blank"}
* Extraer el fichero `sony_dtv0FA50A09A0A9_00004100_155100c1.pkg` y copiarlo a la unidad _flash_
* Introducir la unidad _flash_ en un puerto USB de la televisión
* Al detectar la actualización, la televisión ejecutará el asistente de instalación
* Seguir las instrucciones para copiar el fichero a la memoria de la televisión y actualizarla

> **Nota**: Mientras se realiza el proceso, que dura unos 30 minutos, no hay que retirar la unidad _flash_ hasta que se indique. La televisión se apagará y se reiniciará automáticamente.

Antes de actualizar el _firmware_ la televisión mostraba la siguiente información:

```plaintext
Modelo
Bravia 2015

Versión
7.0

Nivel de parche de seguridad de Android
1 de febrero de 2019

Versión del kernel
3.10.79
root@BuildHost153 #1
Fri Mar 8 22:28:17 JST 2019

Compilación
SVPDTV15_EU-user 7.0 NRD91N.S34 5.433 release-keys
```

Después de la actulización se muestra la siguiente información:

```plaintext
Modelo
Bravia 2015

Versión
7.0

Nivel de parche de seguridad de Android
5 de agosto de 2019

Versión del kernel
3.10.79
root@BuildHost686 #1
Sat Oct 17 08:48:29 JST 2020

Compilación
SVPDTV15_EU-user 7.0 NRD91N.S44 5.457 release-keys
```

Ahora bien, aún habiendo actualizado, la televisión continúa exhibiendo los mismos problemas que comenté anteriormente. Por este motivo he decidido **restablecer a los datos de fábrica** desde el menú de `Ajustes`:

* Ajustes &rarr; Almacenaje y restauración &rarr; Restablecer datos de fábrica
* ¿Restablecer datos de fábrica? **Borrar todo**
* ¿Borrar todo? **Sí**

La televisión ha reiniciado y después de unos minutos ha aparecido la pantalla `Welcome` con el asistente para realizar la configuración inicial.

# Configuración

Los pasos del asistente de instalación son los siguientes:

* Idioma **Español**
* ¿Tienes un teléfono o tablet Android? **No**
* Google: ¿Iniciar sesión en tu cuenta? **Iniciar sesión**

> **Nota**: En este momento es recomendable conectar una teclado en un puerto USB para poder escribir el usuario y contraseña de forma mucho más cómoda que usando el mando a distancia.

* Introduce la dirección de correo electrónico de tu cuenta: `xxxxx.xxxxx@gmail.com`
* Introduce la contraseña de tu cuenta: `xxxxxxxx`
* Google: Condiciones de Servicio y Política de Privacidad: **Aceptar**
* Google: Ubicación? **No**
* Google: Ayuda a mejorar Android TV? **No**
* Seleccione país: **España (Península e Islas Baleares)**
* Bravia: política de privacidad (10/12/2018): _marcar únicamente la primera_ y **Continuar**

```plaintext
[x] Servicios de Sony Smart TV
[ ] Recomendaciones para los programas
[ ] Mejoras de producto
[ ] Publicidad
```

* Bloqueo parental: `xxxx` (_hay que escribirlo dos veces_)
* Desea iniciar la sintonía automática de satélite? **Omitir**^
* Seleccione el tipo de emisión para buscar servicios automáticamente: **Omitir**
* Indique la posición de su TV: **Soporte de sobremesa**
* Hablitar Samba Interactive TV: **Siguiente**
* Samba TV: política de privacidad: **No acepto y desactivar**
* Samba TV: seguro que desea desactivar? **Si**
* Desea cambiar la fecha y la hora actuales? **Omitir** (_son correctas_)
* Configuración finalizada: **Finalizado**

Después de esta configuración inicial, la televisión ha vuelto a funcionar de manera fluída.

# Recuperación de canales

Como en noviembre de 2022 había usado [Sony Channel Editor v1.2.0 (Windows)](https://www.sony.es/electronics/support/lcd-tvs-android-w75xc_w80xc-series/kdl-50w809c/software/00294444){:target="_blank"} para [organizar y borrar los canales de la TDT y el satélite](https://twitter.com/manelrodero/status/1587443954865377284){:target="_blank"} ahora puedo recuperar fácilmente los canales:

> **Nota**: Antes de comenzar el proceso, hay que insertar en un puerto USB una unidad _flash_ que contenga el fichero `sdb.xml` generado por el editor de Sony.

* Ajustes &rarr; Configuración canales &rarr; Configuración digital
* Configuración digital &rarr; Configuración Técnica &rarr; Transferencia de lista de programas &rarr; **Importar**
* La siguiente lista de programas está almacenada en el dispositivo USB:

```plaintext
<País:España>
DVB-T2/DVB-T                <Se importará>
DVB-C <Otros>               No hay servicio
Satélite preferido <Otro>   No hay servicio
Satélite general            <Se importará>
```

* ¿Desea continuar? **Si**
* Esto sobreescribirá la lista de programas de la TV. ¿Desea continuar? **Si**
* Retirar la unidad _flash_ del puerto USB
* Pulsar `Aceptar` para reiniciar

# Aplicaciones

A continuación se configuran las siguientes aplicaciones de _streaming_ (algunas de ellas vienen preinstaladas y otras hay que instalarlas desde [Google Play](https://play.google.com/){:target="_blank"}):

* [Netflix](https://www.netflix.com/es/){:target="_blank"} (preinstalada)
* [Amazon Video](https://www.primevideo.com/){:target="_blank"} (preinstalada)
* [Disney+](https://www.disneyplus.com/es-es){:target="_blank"}
* [HBO Max](https://play.hbomax.com/){:target="_blank"}
