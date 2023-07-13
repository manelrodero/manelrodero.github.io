---
layout: page
title: Hiking 
blog-width: true
---

Esta página contiene información y recursos de utilidad para la práctica del _hiking_ (senderismo, excursionismo) como, por ejemplo, la cartografía digital.

| [Cartografía digital](#cartografía-digital) |

# Cartografía digital

| [IDE](#infraestructuras-de-datos-espaciales) | [Códigos EPSG](#códigos-epsg) | [Niveles de _zoom_](#niveles-de-zoom) | [Documentación](#documentación) | [Desarrollo](#desarrollo) |

## Infraestructuras de Datos Espaciales

A continuación se detallan algunos portales relacionados con las Infraestructuras de Datos Espaciales (IDE) a nivel nacional y autonómico:

### España

* [Instituto Geográfico Nacional](https://www.ign.es/web/ign/portal)
  * [Cartografía y Datos geográficos](http://www.ign.es/web/ign/portal/cbg-area-cartografia)
  * [Servicios web](https://www.ign.es/web/ign/portal/ide-area-nodo-ide-ign)
    * [Cartografía ráster WMS](https://www.ign.es/wms-inspire/mapa-raster?request=GetCapabilities&service=WMS)
    * [Mapa base WMS](https://www.ign.es/wms-inspire/ign-base?request=GetCapabilities&service=WMS)
    * [Ortofotos máxima actualidad del PNOA WMS](https://www.ign.es/wms-inspire/pnoa-ma?request=GetCapabilities&service=WMS)
    * [Cartografía ráster WMTS](https://www.ign.es/wmts/mapa-raster?request=GetCapabilities&service=WMTS)
    * [Mapa base WMTS](https://www.ign.es/wmts/ign-base?request=GetCapabilities&service=WMTS)
    * [Ortofotos máxima actualidad del PNOA WMTS](https://www.ign.es/wmts/pnoa-ma?request=GetCapabilities&service=WMTS)
  * [Visualizador IBERPIX](http://www.ign.es/iberpix/visor)
  * [Canal IGNSpain](https://www.youtube.com/user/IGNSpain) (YouTube)
* [Infraestructura de Datos Espaciales de España](https://www.idee.es/)
  * [Visualizador IDEE](http://www.idee.es/visualizador/)
* [Centro Nacional de Información Geográfica](https://www.cnig.es/)
  * [Centro de Descargas](http://centrodedescargas.cnig.es/)
  * [Fototeca digital](https://fototeca.cnig.es/fototeca/)
* [Sistema Cartográfico Nacional](http://www.scne.es/)

### Catalunya

* [Institut Cartogràfic i Geològic de Catalunya](https://www.icgc.cat/)
  * [Visor VISSIR](http://srv.icgc.cat/vissir/index.html)
  * [Visor de descargas](http://www.icgc.cat/appdownloads/?)
* [Infraestructura de Dades Espacials de Catalunya](https://www.ide.cat/)
* [InstaMaps](https://www.instamaps.cat/#/), hacer un mapa con las dades que se necesiten de forma rápida
  ** [Ruta 105. Sant Llorenç de la Muga a Ermita de Sant Jordi](https://www.instamaps.cat/instavisor/0c22052d364fb8187fc80da2e0b34e57/Ruta_105._Sant_Llorenc_de_la_Muga_a_ermita_de_Sant_Jordi.html)
* [IDE Barcelona](https://www.diba.cat/web/idebarcelona)
* [Geoportal BCN](http://www.bcn.cat/geoportal/ca/presentacio.html)
* [IDE Area Metropolitana de Barcelona](https://www.amb.cat/web/area-metropolitana/dades-espacials)

### Euskadi

* [geoEuskadi](https://www.geo.euskadi.eus/)
  * [Visor geoEuskadi](https://www.geo.euskadi.eus/s69-bisorea/es/x72aGeoeuskadiWAR/index.jsp)
  * [Comparador de Ortofotos](https://www.geo.euskadi.eus/comparador-de-ortofotos/s69-geocont/es/)

### Andalucía

* [IDEAndalucía](https://www.ideandalucia.es/portal/)
  * [Visor IDEAndalucía](http://www.ideandalucia.es/visor/)

## Códigos EPSG

Algunos de los códigos [EPSG](https://spatialreference.org/) (European Petroleum Survey Group) y los Sistemas de Referencia de Coordenadas usados habitualmente son los siguientes:

| Código EPSG | Descripción |
| --- | --- |
| [EPSG:3034](https://epsg.io/3034) | ETRS89-extended / LCC Europe Para la cartografía pan-Europeo a escalas <= 1:500.000 (IDEE) |
| [EPSG:3035](https://epsg.io/3035) | ETRS89-extended / LAEA Europe Para representación y análisis estadístico pan-Europeos (IDEE) |
| [EPSG:3042](https://epsg.io/3042) | ETRS89 / UTM zone 30N (N-E) |
| [EPSG:3395](https://epsg.io/3395) | WGS 84 / World Mercator |
| [EPSG:3587](https://epsg.io/3587) | NAD83(NSRS2007) / Michigan Central |
| [EPSG:3785](https://epsg.io/3785) | Popular Visualisation CRS / Mercator -- Spherical Mercator (deprecated EPSG code wrongly defined) |
| [**EPSG:3857**](https://epsg.io/3857) | WGS 84 / Pseudo-Mercator -- Spherical Mercator, **Google Maps**, OpenStreetMap, Bing, ArcGIS, ESRI |
| [EPSG:4230](https://epsg.io/4230) | Coordenadas Geográficas ED50 |
| [EPSG:4258](https://epsg.io/4258) | Coordenadas Elipsoidales European Terrestrial Reference System 1989 (ETRS89) |
| [EPSG:4267](https://epsg.io/4267) | Coordenadas Geográficas North American Datum 1927 (NAD27) |
| [EPSG:4269](https://epsg.io/4269) | Coordenadas Geográficas North American Datum 1983 (NAD83) |
| [EPSG:4324](https://epsg.io/4324) | Coordenadas Geográficas WGS 72BE Transit Broadcast Ephemeris |
| [EPSG:4326](https://epsg.io/4326) | WGS 84 -- WGS84 - World Geodetic System 1984, used in GPS (**es el utilizado por MOBAC**) |
| [EPSG:23028](https://epsg.io/23028) | Proyección UTM ED50 Huso 28 N |
| [EPSG:23029](https://epsg.io/23029) | Proyección UTM ED50 Huso 29 N |
| [EPSG:23030](https://epsg.io/23030) | Proyección UTM ED50 Huso 30 N |
| [EPSG:23031](https://epsg.io/23031) | Proyección UTM ED50 Huso 31 N |
| [EPSG:25828](https://epsg.io/25828) | Proyección UTM ETRS89 Huso 28 N |
| [EPSG:25829](https://epsg.io/25829) | Proyección UTM ETRS89 Huso 29 N |
| [EPSG:25830](https://epsg.io/25830) | Proyección UTM ETRS89 Huso 30 N |
| [EPSG:25831](https://epsg.io/25831) | Proyección UTM ETRS89 Huso 31 N |
| [EPSG:32628](https://epsg.io/32628) | Proyección UTM WGS84 Huso 28 N |
| [EPSG:32629](https://epsg.io/32629) | Proyección UTM WGS84 Huso 29 N |
| [EPSG:32630](https://epsg.io/32630) | Proyección UTM WGS84 Huso 30 N |
| [EPSG:32631](https://epsg.io/32631) | Proyección UTM WGS84 Huso 31 N |
| [EPSG:900913](https://epsg.io/900913) | Google Maps Global Mercator -- Spherical Mercator (unofficial - used in open source projects / OSGEO) |

En los Servicios Web de Mapas (WMS) el estàndar "de facto" es el EPSG:3857 (WGS 84 / Pseudo-Mercator), también llamado "Web Mercator", "Google Maps Compatible", etc.

## Niveles de _zoom_

El nivel de _zoom_ determina qué parte del mundo es visible en un mapa. Lo más común es proporcionar mapas con 23 niveles de _zoom_, siendo 0 el nivel de zoom más bajo (completamente alejado) y 22 el más alto (completamente acercado).

Cada nivel de _zoom_ está formado por un mosaico de 2^zoom x 2^zoom teselas. Con niveles de zoom bajos, un pequeño conjunto de teselas cubre un área geográfica grande. A niveles de zoom más altos, un mayor número de mosaicos cubre un área geográfica más pequeña.

Inicialmente, en el nivel de _zoom_ **0**, el mapa del mundo cabe en una única tesela. En el nivel **1** se utilizan 4 teselas para representar el mundo en un cuadrado de 2x2 y así sucesivamente:

![Fuente: https://developer.tomtom.com/maps-api/maps-api-documentation/zoom-levels-and-tile-grid][1]

Para descubrir el tamaño real de un solo mosaico en un nivel de zoom dado, podemos usar la fórmula circunferencia del nivel de la tierra / 2^zoom que produce el número de metros por lado del mosaico, donde la circunferencia de la tierra en el Ecuador es igual a 40,075,017 metros.

La mayoría de teselas suelen tener un tamaño de 256x256 píxeles (aunque alguna librería puede trabajar con teselas de 512x512 píxeles) por lo que se puede calcular el número de metros que representa cada una de ellas:

| Nivel de zoom | Metros/píxel | Metros/lado tesela | Núm. de teselas | Escala (pantalla) | Se puede ver |
| --- | --- | --- | --- | --- | --- |
| 0	| 156543 | 40075017 | 1 | 1:500 000 000 | todo el mundo |
| 1	| 78271.5	| 20037508 | 4 | 1:250 000 000 | |
| 2	| 39135.8	| 10018754 | 16 | 1:150 000 000 | área subcontinental |
| 3	| 19567.88 | 5009377.1 | 64 | 1:70 000 000 | país más grande |
| 4	| 9783.94 | 2504688.5 | 256 | 1:35 000 000 | |
| 5	| 4891.97	| 1252344.3 | 1 024 | 1:15 000 000 | gran país africano |
| 6	| 2445.98	| 626172.1 | 4 096 | 1:10 000 000 | gran país europeo |
| 7	| 1222.99	| 313086.1 | 16 384 | 1:4 000 000 | pequeño país, estado de EE. UU. |
| 8	| 611.5	| 156543 | 65 536 | 1:2 000 000 | |
| 9	| 305.75 | 78271.5 | 262 144 | 1:1 000 000 | área amplia, área metropolitana grande |
| 10 | 152.87	| 39135.8 | 1 048 576 | 1:500 000 | área metropolitana |
| 11 | 76.44 | 19567.9 | 4 194 304 | 1:250 000 | ciudad |
| 12 | 38.219 | 9783.94 | 16 777 216 | 1:150 000 | pueblo o distrito de la ciudad |
| 13 | 19.109 | 4891.97 | 67 108 864 | 1:70 000 | pueblo o suburbio |
| 14 | 9.555 | 2445.98 | 268 435 456 | 1:35 000 | |
| 15 | 4.777 | 1222.99 | 1 073 741 824 | 1:15 000 | camino pequeño |
| 16 | 2.3887 | 611.496 | 4 294 967 296 | 1:8 000 | calle |
| 17 | 1.1943 | 305.748 | 17 179 869 184 | 1:4 000 | bloque, parque, direcciones |
| 18 | 0.5972 | 152.874 | 68 719 476 736 | 1:2 000 | algunos edificios, arboles |
| 19 | 0.14929 | 76.437 | 274 877 906 944 | 1:1 000 | detalles de la carretera local y el cruce |
| 20 | 0.14929 | 38.2185 | 1 099 511 627 776 | 1:500 | un edificio de tamaño medio |
| 21 | 0.074646 | 19.10926 | 4 398 046 511 104 | 1:250 | |
| 22 | 0.037323 | 9.55463 | 17 592 186 044 416 | 1:125 | |

Los niveles de _zoom_ que proporcionan diferentes proveedores de mapas son los siguientes:

* [Google](https://developers.google.com/maps/documentation/javascript/maxzoom): mapa de carreteras (hasta el nivel 18); satélite (hasta el nivel 20 o 21)
* [Bing](https://www.bing.com/api/maps/sdk/mapcontrol/isdk/Overview): satélite (hasta el nivel 19)
* VirtualEarth (Satélite): hasta el nivel 19
* [Mapbox](https://docs.mapbox.com/help/getting-started/mapbox-data/): hasta el nivel 22
* [TomTom](https://developer.tomtom.com/maps-api/maps-api-documentation/zoom-levels-and-tile-grid): hasta el nivel 22

## WMTS

Las URL de un servidor WMTS tienen una serie de parámetros estàndard (los nombres de los cuales son _case insensitive_):

| Parámetro | Tipo | Descripción |
| --- | ---| --- |
| SERVICE | Requerido | Tiene que ser "WMTS" |
| VERSION | Opcional | Estándar de la versión WMTS. Por defecto es "1.0.0" |
| REQUEST | Requerido | Qué se pide. Puede ser `GetTile` o `GetCapabilities` |

Cuando se hace una petición `GetTile` se pueden especificar los siguientes parámetros (se pueden saber los diferentes valores mediante la petición `GetCapabilities`):

| Parámetro | Descripción |
| --- | --- |
| TILEMATRIXSET | El conjunto de matrices que se utilizará para el mosaico de salida |
| TILEMATRIX  | La matriz que se utilizará para el mosaico de salida |
| TILECOL | El índice de columna del mosaico de salida |
| TILEROW | El índice de fila del mosaico de salida |
| LAYER | La capa para la que se generará el mosaico de salida |
| FORMAT | El formato de imagen devuelto |

## Google Maps Tiles

Las teselas de Google Maps se pueden obtener mediante la siguiente URL:

[https://mt0.google.com/vt/lyrs=s&hl=es&x={x}&y={y}&z={z}&s=Ga](https://mt0.google.com/vt/lyrs=s&hl=es&x={x}&y={y}&z={z}&s=Ga)

Las diferentes capas (`lyrs`) que se pueden utilizar son:

* h = roads only 0-22
* m = standard roadmap (Plan) 0-22
* p = terrain = t,r 0-22
* r = somehow altered roadmap 0-22
* s = satellite only 0-21
* t = terrain only 0-22
* y = hybrid = s,r (Earth) 0 - 21
* traffic

## Documentación

* [Mapas en la web. Conceptos básicos](http://www.geo.euskadi.eus/cartografia/DatosDescarga/Documentacion/UDA_Ikastaroa_2016/Presentaciones/Conceptos_Basicos.pdf) [PDF]
* [Novedades en los servicios web de visualización de mapas del CNIG](https://www.idee.es/resources/presentaciones/JIIDE14/20141106/WMTS_CNIG_presentacion.pdf) [PDF]
* [Cómo incluir servicios WMTS y WMS en la API de Google](http://www.ign.es/resources/viewer/APIGoogle_incluirwms_wmts.pdf) [PDF]
* [¿Cómo interpretar un Mapa Topográfico?](https://youtu.be/Uy539-fLofs) [YouTube]
* [Zoom levels and scale](https://developers.arcgis.com/documentation/mapping-apis-and-services/reference/zoom-levels-and-scale/)
* [Esquema de teselado "GoogleMapsCompatible TileMatrixSet"](https://idearm.imida.es/aet2017/contenidos/salaB/jueves/02.GuilermoVilla-2017-10-04%20Villa%20et%20al%20Esquema%20de%20Teselado.pdf) [PDF]
* [Creating the “Perfect” Hiking Map for Germany and other Countries](https://projects.webvoss.de/2017/06/03/creating-the-perfect-hiking-map-for-germany-and-other-countries/)
* [Afegeix mapes ICGC a Google Earth](https://www.icgc.cat/Administracio-i-empresa/Serveis/Geoinformacio-en-linia-Geoserveis/Serveis-per-a-aplicacions-d-escriptori/Afegeix-mapes-ICGC-a-Google-Earth)
* [Google Earth Pro, como utilizar capas de mapas con Web Map Service](https://www.latrencanous.com/google-earth-pro-como-utilizar-capas-de-mapas-con-web-map-server-wms/)

## Blogs

* [Geotics](https://juanchosierrar.blogspot.com/): Blog de Juan Carlos Sierra con bastantes artículos y vídeos sobre Google Earth (canal de [YouTube](https://www.youtube.com/channel/UCjwQKkrMSpYYw4Zt57uZ06Q))

## Software

* [Google Earth Studio](https://www.google.com/earth/studio/)
* [Garmin BaseCamp](https://www.garmin.com/es-ES/software/basecamp/)
* [GPS Visualizer](https://www.gpsvisualizer.com/): Herramienta online que permite crear un mapa y perfiles a partir de datos geográficos. Se pueden introducir datos GPS (tracks y waypoints), coordenadas, rutas de coche, etc.
* [uTrack](http://utrack.crempa.net/index_es.php): Generador de informes online de tracks GPX (estadísticas de distancia y elevación, mapa, etc.). **Obsoleto**
* [GPX2KML](https://gpx2kml.com/): Conversor entre ficheros GPX/KML (waypoints, tracks, routes)
* [KML2GPX](https://kml2gpx.com/): Conversor entre ficheros GPX/KML (waypoints, tracks, routes)

## Desarrollo

* [Keyhole Markup Language](https://developers.google.com/kml/documentation): KML es un formato de archivo que se utiliza para mostrar datos geográficos en programas como Google Earth. Puede crear archivos KML para identificar ubicaciones, agregar superposiciones de imágenes y exponer datos enriquecidos de nuevas formas. KML es un estándar internacional mantenido por Open Geospatial Consortium, Inc. (OGC). El [KML Tutorial](https://developers.google.com/kml/documentation/kml_tut) es la forma más fácil de aprender a trabajar con este tipo de archivos.

* [OpenLayers](https://openlayers.org/): A high-performance, feature-packed library for all your mapping needs
* [MapTiler](https://www.maptiler.com/): Mapping platform for quick publishing of zoomable maps online
* [epsg.io](https://epsg.io/): Coordinate Systems Worldwide
* [TomTom Map Display API](https://developer.tomtom.com/maps-api)
* [Mapbox](https://www.mapbox.com/): Maps and location for developers
* [MapServer](https://mapserver.org/es/index.html): Open Source platform for publishing spatial data and interactive mapping applications to the web
* [Google Map Tile Info](https://glennmessersmith.com/maps/tileinfo.htm)

## FAQs

## Track vs Tour (Animated) vs Path

Al cargar un fichero KML en Google Earth puede pasar que aparezca un **control deslizante de tiempo** debido a que el fichero contiene **marcas de tiempo** asociadas a la ruta de navegación GPS (campos `<when>`):

* Un **track** está formado por un `<gx:Track>` con muchos campos `<when>`
* Un **path** está formado por un `<linestring>` con `<coordinates>`

Se pueden eliminar estas marcas de tiempo copiando el _track_ del panel de navegación izquierdo de Google Earth y pegándolo en un editor de textos que soporte expresiones regulares (como Notepad++) para borrar todos los valores `<when>`:

* Copiar el _track_ del panel de navegación izquierdo de Google Earth
* Pegar el texto en Notepad++ o abrir el fichero KML
* Abrir el diálogo para buscar y reemplazar (`Ctrl+H`)
* Marcar la opción para usar **expresiones regulares** y utilizar la siguiente expresión en la búsqueda: `<when>.*</when>`
* Reemplazar todos los valores por **vacío** (_empty_)

[1]: /assets/img/hobbies-hiking_image_1.png "Fuente: https://developer.tomtom.com/maps-api/maps-api-documentation/zoom-levels-and-tile-grid"
