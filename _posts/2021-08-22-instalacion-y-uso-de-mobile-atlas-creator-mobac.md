---
layout : post
blog-width: true
title: 'Instalación y uso de Mobile Atlas Creator (MOBAC)'
date: '2021-08-22 14:28:42'
published: true
tags:
- GPS
author:
  display_name: Manel Rodero
---

[Mobile Atlas Creator](https://mobac.sourceforge.io/) es un programa de código abierto (GPL) que crea atlas sin conexión para dispositivos GPS y aplicaciones de teléfonos móviles Android e iOS (entre ellos el conocido [OruxMaps](https://www.oruxmaps.com/cs/es)). Además, los mapas individuales se pueden exportar como una imagen PNG grande con un archivo MAP de calibración para [OziExplorer](https://www.oziexplorer4.com/w/).

Como fuente para un atlas fuera de línea, Mobile Atlas Creator puede utilizar una gran cantidad de mapas en línea diferentes, como mapas basados en OpenStreetMap y otros proveedores de mapas en línea ([Instituto Geográfico Nacional](https://www.ign.es/web/ign/portal), [Institut Cartogràfic i Geològic de Catalunya](https://www.icgc.cat/), etc.).

* [IGN Base WMTS (Spain)](https://www.ign.es/wmts/ign-base?request=GetCapabilities&service=WMTS) - Mapa base de España
> Capa **IGNBaseTodo**: Representación cartográfica desde escalas pequeñas hasta 1:4.000 de nombres geográficos, redes de transporte y sus infraestructuras, espacios naturales, culturales y arqueológicos, hidrografía, relieve, núcleos urbanos, islas, manzanas, edificios y direcciones, junto con la representación del mar y los límites de los países.
* [IGN MTM raster WMTS (Spain)](https://www.ign.es/wmts/mapa-raster?request=GetCapabilities&service=WMTS) - Cartografía raster de España del IGN
> Capa **MTN**: Cartografía raster del IGN donde se muestran diferentes mapas en función de la escala de visualización: Mapa 2M (hasta 305m/px). Mapa 1M (hasta 152 m/px), Mapa 500 (hasta 50 m/px), Mapa 200 (hasta 20 m/px), MTN50 (hasta 5 m/px), MTN25 (desde 5 m/px).
* [IGN PNOA WMTS (Spain)](https://www.ign.es/wmts/pnoa-ma?request=GetCapabilities&service=WMTS) - Ortoimágenes de España (satélite Sentinel2 y ortofotos del PNOA máxima actualidad)
> Capa **OI.OrthoimageCoverage**: Imagen de satélite Sentinel2 a escalas menores de 1:70.000 y las ortofotografías PNOA de máxima actualidad para escalas mayores, para toda España.

Estas fuentes de datos utilizan el estandar [**Web Map Tile Service (WMTS)**](https://en.wikipedia.org/wiki/Web_Map_Tile_Service), una variante del [Web Map Service (WMS)](https://es.wikipedia.org/wiki/Web_Map_Service) optimizada para servir _tiles_ (teselas) de mapas georeferenciadas.

Una **tesela** es un recorte o pieza de tamaño definido de un mapa que puede implementarse a diversas **escalas** y mostrarse de forma dinámica (los metros que representa cada tesela están definidos según el nivel de _zoom_ del mapa):

![Fuente: https://nieneb.github.io/aeres_workshop/][1]

## Requerimientos

Mobile Atlas Creator está escrito en Java y, por lo tanto, requiere [Java Runtime Environment](https://www.java.com/es/download/) (JRE) versión 8 o superior para funcionar. Sin embargo, la versión de Java recomendada para ejecutar MOBAC es [OpenJDK](https://openjdk.java.net/) versión 11 o superior:

* [OpenJDK](https://jdk.java.net/16/)
  * [openjdk-11.0.2_windows-x64_bin.zip](https://download.java.net/java/GA/jdk11/9/GPL/openjdk-11.0.2_windows-x64_bin.zip)
  * [openjdk-16.0.2_windows-x64_bin.zip](https://download.java.net/java/GA/jdk16.0.2/d4a915d82b4c4fbb9bde534da945d746/7/GPL/openjdk-16.0.2_windows-x64_bin.zip)
* [Oracle Java SE](https://www.oracle.com/java/technologies/javase-downloads.html)
  * [jdk-11.0.12_windows-x64_bin.zip](https://www.oracle.com/webapps/redirect/signon?nexturl=https://download.oracle.com/otn/java/jdk/11.0.12%2B8/f411702ca7704a54a79ead0c2e0942a3/jdk-11.0.12_windows-x64_bin.zip)
  * [jdk-16.0.2_windows-x64_bin.zip](https://download.oracle.com/otn-pub/java/jdk/16.0.2%2B7/d4a915d82b4c4fbb9bde534da945d746/jdk-16.0.2_windows-x64_bin.zip)
* [Adoptium OpenJDK](https://adoptium.net/releases.html)
  * [OpenJDK11U-jdk_x64_windows_hotspot_11.0.12_7.zip](https://github.com/adoptium/temurin11-binaries/releases/download/jdk-11.0.12%2B7/OpenJDK11U-jdk_x64_windows_hotspot_11.0.12_7.zip)
  * [OpenJDK16U-jdk_x64_windows_hotspot_16.0.2_7.zip](https://github.com/adoptium/temurin16-binaries/releases/download/jdk-16.0.2%2B7/OpenJDK16U-jdk_x64_windows_hotspot_16.0.2_7.zip)

En este caso se descarga el fichero `jdk-16.0.2_windows-x64_bin.zip` correspondiente a Oracle Java SE. El contenido del fichero ZIP se extrae en el directorio `E:\Cloud\OneDrive\SOFTWARE\Java\jdk-16.0.2` y queda listo para ser utilizado.

## Instalación

A día de hoy, la última versión disponible es la **2.2.1** (2021-06-02) que se puede descargar desde la página del proyecto en [Sourceforge](https://sourceforge.net/projects/mobac/).

La "instalación" es muy sencilla: únicamente es necesario extraer el contenido del fichero `Mobile Atlas Creator 2.2.1.zip` en el directorio que se desee (p.ej. `E:\Cloud\OneDrive\SOFTWARE\MOBAC`).

A continuación se puede ejecutar utilizando la versión de Java que se ha descargado anteriormente:

```
..\Java\jdk-16.0.2\bin\java.exe -Xms64m -Xmx1200M -jar Mobile_Atlas_Creator.jar
```

## Configuración inicial

Al ejecutarlo por primera vez, hay que introducir los datos para crear el primer **Atlas**. En este ejemplo, se utilizará el formato **OruxMaps Sqlite** que servirá para añadir los mapas a la aplicación [OruxMaps](https://play.google.com/store/apps/details?id=com.orux.oruxmapsDonate) para Android:

* Nombre del atlas: `Test`
* Formato del atlas: `OruxMaps Sqlite`

Antes de crear un mapa, se ajustan los siguientes parámetros en `Tools > Settings`:

* Map sources config > Perform online update (y reiniciar el programa)
* Map sources (deshabilitar todos los mapas excepto `OpenStreetMap 4UMaps.eu` y los tres correspondientes al IGN)
* Map size: 1048575
* Directories > Atlas output directory > `D:\USUARIS\Manel\Documents\GPX\Atlases`
* Network > Network connections > 6
* Network > Default > Ignore download errors and continue automatically

## Crear un mapa

A la hora de crear un mapa es muy importante utilizar una _grid_ del nivel _zoom_ adecuado para seleccionar las teselas que se descargarán. Por ejemplo, un **Grid Zoom 8** está bastante bien para seleccionar en mapas _raster_ o topográficos. En un mapa satélite se puede utilizar el **Grid Zoom 9** para seleccionar zonas más pequeñas. Si la zona a descargar es pequeña, se puede seleccionar usando la rejilla 12 (menos zona) y así descargar el nivel 18 para tener más detalle sin que el mapa ocupe demasiado.

Si se carga la [cartografía raster de España del IGN](https://www.ign.es/wmts/mapa-raster?request=GetCapabilities&service=WMTS) en el [visualizador del IDEE](https://www.idee.es/visualizador/) se puede observar la equivalencia entre los [niveles de zoom y la escala del mapa](/hobbies/hiking/#niveles-de-zoom):

| Nivel de zoom | Escala pantalla | Mapa papel |
| --- | --- |
| 10 | 1:441238 | 1:500000 |
| 11 | 1:220119 | 1:250000 |
| 12 | 1:109927 | 1:125000 |
| 13 | 1:54930 | 1:50000 |
| 14 | 1:27454 | 1:25000 |
| 15 | 1:13725 | 1:15000 |
| 16 | 1:6804 | 1:10000 |
| 17 | 1:3402 | 1:5000 |
| 18 | 1:1701 | 1:2500 |

El **zoom 14** (1:25000) es el mapa que se suele utilizar en los GPS. Los zooms más lejanos se utilizan para moverse rápido por el mapa y los zooms más cercanos para seleccionar zonas del mismo.

Los zooms 15, 16 y 17 son básicamente el mismo mapa. Hay pequeñas diferencias en la calidad y definición de los mismos pero bastante diferencia en el tamaño de las teselas que se descargan. Por tanto, se puede descargar el **zoom 16** para mantener un tamaño razonable.

> **Nota**: cuando OruxMaps descarga un mapa, los niveles que no se han descargado se construyen utilizando el mapa de mayor tamaño (perdiendo calidad). Esto es necesario para que haya una transición suave al hacer zoom en el mapa.

En las [ortoimágenes de España](https://www.ign.es/wmts/pnoa-ma?request=GetCapabilities&service=WMTS) no tiene sentido descargar los niveles 11, 12, 13, etc. Es muy común utilizar el **zoom 17** (aprox. 1:5000). Si la zona a descargar es muy pequeña, se puede utilizar la rejilla 12 para seleccionar y descargar el **zoom 18** (aprox. 1:2500)

> **Nota**: Si se descarga un mapa satélite (ortofoto) hay veces en que puede ser interesante tener encima del mismo un mapa vectorial para poder ver los caminos que tapan los árboles (si se está descargando un mapa topográfico no se hará porque se duplicaría la información).

Hacer un mapa para un track; cargando un GPX
Añadir selección around gpx
Distancia alrededor -> 238m
Mostrar zonas seleccionadas

Google Earth:
- Cargar o crear un track
- Añadir places/waypoints (desde altura 1,7km)
- Exportar a KML para cargarlo en OruxMaps

ML2GPX.com
- Cargar KML y exportar Tracks a GPX

ORUXMAPS
- Cargar GPX y crear mapa


## Agregar fuentes de datos

Se pueden agregar nuevas fuentes de datos creando un fichero `*.xml` en el directorio `mapsources` de la aplicación. En este fichero XML se define la URL del servicio WMTS y otros datos para acceder a las teselas.

> **Nota**: Se puede utilizar el [**visualizador del IDEE**](https://www.idee.es/visualizador/) para probar la URL del servicio, ver las capas existentes y sus nombres, etc. antes de agregarlo a MOBAC. El [**visor Iberpix**](http://www.ign.es/iberpix/visor) también puede ser una buena ayuda para obtener esta información.

Un ejemplo para descargar las teselas de la cartografía ráster del IGN sería el siguiente:

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>

<!--
Mapa   : Cartografía Ráster de España del IGN
URL    : https://www.ign.es/wmts/mapa-raster?request=GetCapabilities&service=WMTS
Fuente : https://www.ign.es/web/ign/portal/ide-area-nodo-ide-ign
Fecha  : 2021-08-22
-->

<customMapSource>
  <!-- https://mobac.sourceforge.io/wiki/index.php/Custom_XML_Map_Sources#customMapSource -->

  <!-- Nombre del mapa que aparecerá en el programa -->
  <name>Custom 02 IGN-Raster WMTS (ES)</name>

  <!--- Niveles de zoom que permitirá hacer el programa -->
  <minZoom>5</minZoom>
  <maxZoom>20</maxZoom>

  <!--- Formato de imagen proporcionado por el servidor -->
  <tileType>JPG</tileType>

  <!-- Si la tesela ha expirado, se vuelve a descargar -->
  <tileUpdate>None</tileUpdate>

  <!--- URL para obtener las teselas del servidor WMTS -->
  <url><![CDATA[https://www.ign.es/wmts/mapa-raster?layer=MTN&style=default&tilematrixset=GoogleMapsCompatible&Service=WMTS&Request=GetTile&Version=1.0.0&Format=image%2Fjpeg&TileMatrix={$z}&TileCol={$x}&TileRow={$y}]]></url>

  <!-- Color de las teselas inexistentes -->
  <backgroundColor>#000000</backgroundColor>
</customMapSource>
```

También es posible utilizar ficheros `*.bsh` (Beanshell) para definir las URL desde las que se descargan las teselas.

Un ejemplo para descargar las teselas desde Google en formato satélite sería el siguiente:

```
// Mapa   : Google Satélite
// URL    : 
// Fuente : 
// Fecha  : 2021-08-22

// Nombre del mapa en MOBAC
name = "Custom BSH Google Satelite";

// UserAgent y Referer para añadir a las cabeceras
String MyUserAgent = "Mozilla/5.0 Gecko/20100101 Firefox/49.0";
String MyReferer = "";

// Parámetros del servidor
String Lyrs = "s";
tileType = "jpg";
maxZoom = 21;

/**
h = roads only 0-22
m = standard roadmap (Plan) 0-22
p = terrain = t,r 0-22
r = somehow altered roadmap 0-22
s = satellite only 0-21
t = terrain only 0-22
y = hybrid = s,r (Earth) 0.21
**/

String getTileUrl( int Zoom, int X, int Y ) {
	return "http://mt0.google.com/vt/lyrs="+Lyrs+"&hl=fr_FR&x="+X+"&y="+Y+"&z="+Zoom;    
} 

void addHeaders(java.net.HttpURLConnection conn) 
{
	conn.addRequestProperty("Referer",MyReferer);
	conn.addRequestProperty("User-Agent",MyUserAgent);	
}
```

## ¿Cómo obtener la URL WMTS?

Obtener la URL que obtiene una tesela de un servidor WMTS es un procedimiento bastante sencillo si se utiliza **Google Chrome**:

* Acceder al [visualizador del IDEE](https://www.idee.es/visualizador/)
* Aquí hay dos opciones:
  * Cargar la capa de información geográfica de entre las disponibles
  * Introducir la URL para obtener las capacidades del servicio WMTS `https://<servidor-wmts>?request=GetCapabilities&service=WMTS`
* Abrir las herramientas de depuración mediante `F12`
* Seleccionar la opción `Network`
* Recargar la página mediante `CTRL+R`
* Copiar la URL que obtiene una tesela
* Sustituir en la URL las [variables](https://mobac.sourceforge.io/wiki/index.php/Custom_XML_Map_Sources#url) _zoom_ `{$z}`, coordenada x `{$x}`y coordenada y `{$y}`

## Referencias

* [Estándares WMS, WMTS, WFS y WCS del OGC: qué son y diferencias](http://www.geomapik.com/webmapping-gis/estandares-ogc-wms-wmts-wfs-wcs/) @ geomapik
* [Utilisation de Mobac (Mobile Atlas Creator)](http://randochartreuse.free.fr/mobac2.x/)
* [Liste de fichiers BSH ou XML prêts à l'emploi pour Mobac](http://randochartreuse.free.fr/mobac2.x/mapsources/)
* [MOBAC .bsh Map Files](https://fluidicice.com/blog/?p=904)
* [Cartografía Digital](https://www.cartografiadigital.es/)
* [Mapas WMS, GPS y Mobile Atlas Creator (MOAC)](http://www.multicopterox.es/mapas-wms-gps-y-mobile-atlas-creator-moac/)
* [Mapas offline para dispositivos móviles, Mobac](https://www.bytacora.es/sig-2/creacion-de-mapas-offline-para-dispositivos-moviles-mobac/index.html)
* [MOBAC: mapas para OruxMaps](https://youtu.be/nauVWH8LIDw): Vídeo de [Cartografía Digital](cartografiadigital.es)
* [Crear mapas para Twonav con Mobac - 2020](https://youtu.be/ZqZ1icwGJuA): Video de Antonio Ruíz ([Perchera.com](https://perchera.com/tutoriales/crear-mapas-con-mobac/))

<p></p>

[1]: /assets/img/blog/2021-08-22_image_1.png "Fuente: https://nieneb.github.io/aeres_workshop/"
