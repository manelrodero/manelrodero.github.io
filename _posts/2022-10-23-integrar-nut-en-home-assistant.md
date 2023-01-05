---
layout : post
blog-width: true
title: 'Integrar NUT en Home Assistant'
date: '2022-10-23 16:31:08'
published: true
tags:
- Home Assistant
- Raspberry
author:
  display_name: Manel Rodero
---

En los artículos anteriores he explicado cómo [monitorizar un SAI usando **NUT**](monitorizar-un-sai-ups-desde-una-raspberry-pi) y cómo [instalar **Home Assistant** usando Docker](instalacion-de-home-assistant-en-docker).

En este pequeño artículo explicaré cómo se ha configurado la [**integración** de NUT en Home Assistant](https://www.home-assistant.io/integrations/nut) que ha sido descubierto de forma automática.

* Acceder a la instancia de Home Assistant
* En la barra lateral hacer clic en `Settings`
* En el menú de configuración seleccionar `Devices and Services`
* En el menú superior seleccionar `Integrations`
* Seleccionar la integración `Network UPS Tools (NUT)`

Esta integración dispone, en mi caso, de **1 dispositivo** y **18 entidades**. Al hacer clic en el dispositivo se observa, obviamente, el MGE Protection Center 750 que se configuró anteriormente.

Las **entidades** disponibles están divididas en **sensores** y **diagnósticos** y, en este caso, se han habilitado las mostradas en la siguiente imagen:

![Sensores y Diagnósticos][1]

* Sensores
  * Batería cargada (porcentaje)
  * Carga (porcentaje)
  * Tensión de salida (Volts)
  * Estado (Online, Offline)
* Diagnóstico
  * Duración de batería (segundos)

Además, se ha creado una **tarjeta** de tipo entidades que se ha agregado a un **tablero** ([_**dashboard**_](https://www.home-assistant.io/dashboards)) llamado `Casa`:

![Tarjeta SAI][2]

> Debido a que el nombre en clave del desarrollo de los tableros era _lovelace_, aún se pueden encontrar referencias a este nombre en tutoriales y vídeos.

## Mejoras

Como la duración de la batería se expresa en segundos, se puede crear un nuevo **sensor** usando [**plantillas**](https://www.home-assistant.io/integrations/template/) para convertirlo a minutos.

En el fichero de configuración de Home Assistant (`/config/configuration.yaml`) se añade una línea para incluir un fichero con los sensores:

```
sensor: !include sensors.yaml
```

A continuación se crea el fichero `/config/sensors.yaml` con el siguiente contenido:

```yaml
{%raw%}
- platform: template
  sensors:
    mge750_battery_runtime_min:
      friendly_name: Battery Runtime
      unit_of_measurement: 'min'
      value_template: >-
        {{ states("sensor.mge750_battery_runtime") | int // 60 }}
      icon_template: mdi:timer-outline
{%endraw%}
```

Esta plantilla crea un sensor llamado `mge750_battery_runtime_min`, medido en minutos, que está basado en el sensor original `mge750_battery_runtime`, medido en segundos.

Este sensor tiene como valor el resultado de dividir el valor del sensor original por 60 [convertido en un número entero](https://jinja.palletsprojects.com/en/latest/templates/#math) usando [**Jinja**](https://jinja.palletsprojects.com/en/latest/templates/).

Se puede hacer lo mismo para aproximar el consumo de energía usando el porcentaje de carga y la capacidad nominal del SAI (en este caso 750 W):

```yaml
{%raw%}
- platform: template
  sensors:
    mge750_approx_power:
      friendly_name: Approximate Power
      unit_of_measurement: 'W'
      value_template: >-
        {{ states("sensor.mge750_load") | int / 100 * 750 | round(1, "floor") }}
      device_class: power
{%endraw%}
```

Después de comprobar que la sintaxis es correcta, se puede reiniciar Home Assistant:

* Developer Tools > YAML > Check Configuration
* Developer Tools > YAML > Restart

y añadir las nuevas entidades al tablero para que quede algo así:

![Custom Sensors][3]

## Referencias

* [PowerWalker UPS, NUT, and Home Assistant](https://blog.cavelab.dev/2022/01/powerwalker-nut-home-assistant/)
* [How to set up custom sensors in Home Assistant](https://opensource.com/article/21/2/home-assistant-custom-sensors)
* [Home Assistant template sensor, adding two sensor explained](https://blogit.create.pt/ricardocosta/2020/10/28/home-assistant-template-sensor-adding-two-sensor-explained/)

[1]: /assets/img/blog/2022-10-23_image_1.png "Sensores y Diagnósticos"
[2]: /assets/img/blog/2022-10-23_image_2.png "Tarjeta SAI"
[3]: /assets/img/blog/2022-10-23_image_3.png "Custom Sensors"
