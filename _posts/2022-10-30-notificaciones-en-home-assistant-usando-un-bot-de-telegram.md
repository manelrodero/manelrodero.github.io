---
layout : post
blog-width: true
title: 'Notificaciones en Home Assistant usando un bot de Telegram'
date: '2022-10-30 13:47:27'
published: true
tags:
- Home Assistant
author:
  display_name: Manel Rodero
---

Después de [instalar Home Assistant en Docker](instalacion-de-home-assistant-en-docker) puede ser interesante configurar la integración con [**Telegram**](https://www.home-assistant.io/integrations/telegram/){:target="_blank"} para recibir notificaciones con los cambios que se producen en casa a través de las automatizaciones.

{: .box-note}
No confundir con la integración [Telegram bot](https://www.home-assistant.io/integrations/telegram_bot/){:target="_blank"} que permite enviar órdenes desde Telegram a Home Assistant para que ejecute diferentes acciones (por ejemplo, poner en marcha la calefacción).

# Crear un bot en Telegram

Para poder realizar esta integración es necesario [crear un bot de Telegram](https://core.telegram.org/bots/features) que será el encargado de interactuar con Home Assistant.

La forma más fácil de crear un bot es usar la aplicación **@BotFather**, el propio bot de Telegram que permite crearlos de forma sencilla usando el comando `/newbot`:

![BotFather #1][1] ![BotFather #2][2]

El procedimiento para hacerlo es el siguiente:

* Abrir una conversación con BotFather accediendo a [https://telegram.me/botfather](https://telegram.me/botfather)
* Pulsar en el botón `Start` para comenzar y ver los comandos que hay disponibles
* Enviar el comando `/newbot`
* Indicar el nombre del bot, por ejemplo `Notificaciones HA`
* Indicar el _username_ del bot, por ejemplo `notifyhabot`

A continuación BotFather devolverá un **token** similar al mostrado a continuación que permitirá utilizar la [API HTTP de Telegram](https://core.telegram.org/bots/api):

```
6324140421:BB024hadfsa_13421jñqleEEjasgGGMadfD
```

> **Nota**: hay que guardar este _token_ como cualquier otra contraseña, de manera segura, ya que es la llave que permite controlar el bot.

Este bot puede tener un perfil como el de cualquier usuario, incluyendo un imagen, una descripción, etc. Además, se podrá definir cómo se comportará al recibir determinados comandos.

Como los bots no pueden contactar con los usuarios, hay que enviarle un primer mensaje desde nuestro usuario de Telegram:

* Buscar el bot que acabamos de crear (en este ejemplo era [https://telegram.me/notifyhabot](https://telegram.me/notifyhabot))
* Enviarle un primer mensaje (p.ej `test`) desde nuestro usuario
* Acceder a la API de Telegram indicando el **token** obtenido anteriormente usando una URL similar a la siguiente:

```
https://api.telegram.org/bot<API_TOKEN>/getUpdates
```

Si todo es correcto, se obtendrá una respuesta en formato `*.json` que contiene, entre otras cosas, el **chat_id** de la conversación con nuestro usuario:

```json
{
    "ok": true,
    "result": [
        {
            "update_id": 254199982,
            "message": {
                "message_id": 3,
                "from": {
                    "id": 123456789,
                    "is_bot": false,
                    "first_name": "First",
                    "last_name": "Last",
                    "username": "firstlast",
                    "language_code": "es"
                },
                "chat": {
                    "id": 123456789,
                    "first_name": "First",
                    "last_name": "Last",
                    "username": "firstlast",
                    "type": "private"
                },
                "date": 1667147343,
                "text": "test"
            }
        }
    ]
}
```

# Asociar el bot a Home Assistant

A continuación se puede editar el fichero `configuration.yaml` para [conectar el bot con Home Assistant](https://www.home-assistant.io/integrations/telegram_bot/) usando el **token** y el **chat_id** obtenidos anteriormente:

```yaml
telegram_bot:
  - platform: polling
    api_key: "6324140421:BB024hadfsa_13421jñqleEEjasgGGMadfD"
    allowed_chat_ids:
      - 123456789
```

y añadir un elemento **notificador** que será el encargado de enviar las notificaciones al chat entre el usuario y el bot:

```yaml
notify:
  - platform: telegram
    name: notifyhabot
    chat_id: 123456789
```

Después de guardar los cambios en el fichero, se reinicia Home Assistant y todo estará preparado para utilizar este notificador en las automatizaciones.

# Probar las notificaciones

La prueba que se realizará es una acción muy sencilla. Vamos a intentar que, cuando el precio de la luz sea menor que 0,27€/kWh, se envíe una notificación a nuestro Telegram.

Para ello se creará una automatización que use la integración [Spain electricity hourly pricing (PVPC)](https://www.home-assistant.io/integrations/pvpc_hourly_pricing/) que instalé hace unos días.

* Settings > Automation & Scenes
* Create Automation
* Start with an empty automation
* Add Trigger > `Numeric state`
  * Entity > `sensor.pvpc`
  * Below > `0.27`
* Add Action > `Call service`
  * Service > `notify.notifyhabot`
  * Title > `Precio de la luz`
  * Message > `Por debajo de 0.27€/kWh`
* Save
* Name: `Aviso precio luz`

Cuando el valor del sensor esté por debajo de 0,27 se recibirá un aviso en Telegram como el mostrado en la siguiente captura:

![Mensaje Telegram][3]

# Referencias

* [Telegram Bot Features](https://core.telegram.org/bots/features)
* [HA: Telegram](https://www.home-assistant.io/integrations/telegram)
* [HA: Telegram bot](https://www.home-assistant.io/integrations/telegram_bot/)
* [Home Assistant: Send Telegram Notifications via Automations](https://theinad.com/posts/2022-01-26/home-assistant-send-telegram-notifications-via-automations/)
* [Enviando alertas de Telegram con Home Assistant o Hassio](https://www.bujarra.com/enviando-alertas-de-telegram-con-home-assistant-o-hassio/)
* [Telegram Channel vs Group: Which One Should You Use](https://www.guidingtech.com/telegram-channel-vs-group-which-one-should-you-use/)

[1]: /assets/img/blog/2022-10-30_image_1.png "BotFather #1"
[2]: /assets/img/blog/2022-10-30_image_2.png "BotFather #2"
[3]: /assets/img/blog/2022-10-30_image_3.png "Mensaje Telegram"
