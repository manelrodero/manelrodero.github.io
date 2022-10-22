---
layout : post
blog-width: true
title: 'Monitorizar un SAI (UPS) desde una Raspberry Pi'
date: '2022-10-21 19:44:05'
published: true
tags:
- Raspberry
author:
  display_name: Manel Rodero
---

Un **SAI** (Sistema de Alimentación Ininterrumpida), también conocido por sus siglás in inglés **UPS** (_Uninterrumpible Power Supply_), es un aparato o dispositivo que tiene como misión proporcionar corriente en caso de corte o problema eléctrico. 

En este artículo se explica como monitorizar uno, concretamente un **[MGE Protection Center 750](http://powerquality.eaton.com/about-us/mgeops.asp)** similar al mostrado en la imagen siguiente, usando el proyecto [**Network UPS Tools**](https://networkupstools.org/) (`nut`).

![MGE Protection Center 675][1]

## Conexión USB

El primer paso para poder monitorizarlo es conectar el SAI a la Raspberry Pi por USB utilizando el cable adaptador [RJ45 a USB](https://networkupstools.org/docs/user-manual.chunked/apgs03.html#_mge_office_protection_systems) incluido con el aparato.

Una vez conectado, se puede ejecutar el comando `lsusb` para listar los dispositivos USB reconocidos:

```
lsusb
```
> Si el comando anterior no existe, será necesario instalar el paquete `usbutils` mediante `apt-get`.

Entre los diferentes dispositivos USB que tenga conectados la Raspberry Pi debería aparecer el SAI de MGE:

```
Bus 001 Device 004: ID 0463:ffff MGE UPS Systems UPS
```

## Instalación de `nut`

El proyecto `nut` soporta diferentes [configuraciones](https://networkupstools.org/docs/user-manual.chunked/ar01s03.html#_monitoring_diagrams) de monitorización.


En estas instrucciones se instalará y configurará para funcionar en el modo más simple (**standalone**) donde toda la comunicación se produce entre el SAI y la Raspberry Pi:

![Esquema de monitorización][2]

En este modo de funcionamiento, el _driver_ para comunicarse con el SAI, el cliente `upsmon` y el servidor `upsd` se ejecutan en la Raspberry Pi.

Para instalar estos componentes hay que ejecutar el siguiente comando `apt-get`:

```
sudo apt-get update && sudo apt-get install nut nut-client nut-server
```

## Configuración del _driver_

Una vez instalados todos los componentes, el primer paso es realizar la configuración del _driver_ para que éste se comunique con el SAI.

Según la [Hardware Compatibility List](https://networkupstools.org/stable-hcl.html) del proyecto `nut`, para este modelo concreto se debe utilizar el _driver_ genérico [`usbhid-ups`](https://networkupstools.org/docs/man/usbhid-ups.html).

Por tanto, se edita el fichero [`ups.conf`](https://networkupstools.org/docs/man/ups.conf.html):

```
sudo vi /etc/nut/ups.conf
```

y se añade una configuracion para este SAI indicando un nombre identificativo (en este caso `mge750`), el _driver_ que se utilizará, el puerto que usará para comunicarse (automático) y una pequeña descripción:

```
[mge750]
        driver = usbhid-ups
        port = auto
        desc = "MGE Protection Center 750"
```

A continuación se pone en marcha el `nut-driver` mediante el siguiente comando:

```
sudo systemctl start nut-driver
```

y se comprueba que se inicia correctamente:

```
sudo systemctl status nut-driver
```
```
● nut-driver.service - Network UPS Tools - power device driver controller
   Loaded: loaded (/lib/systemd/system/nut-driver.service; static; vendor preset: enabled)
   Active: inactive (dead) since Wed 2022-10-19 21:52:25 CEST; 18s ago
  Process: 17943 ExecStart=/sbin/upsdrvctl start (code=exited, status=0/SUCCESS)
  Process: 17949 ExecStop=/sbin/upsdrvctl stop (code=exited, status=0/SUCCESS)
 Main PID: 17948 (code=exited, status=0/SUCCESS)

Oct 19 21:52:21 pi4nas upsdrvctl[17943]: Network UPS Tools - Generic HID driver 0.41 (2.7.4)
Oct 19 21:52:21 pi4nas upsdrvctl[17943]: USB communication driver 0.33
Oct 19 21:52:23 pi4nas upsdrvctl[17943]: Network UPS Tools - UPS driver controller 2.7.4
Oct 19 21:52:23 pi4nas usbhid-ups[17948]: Startup successful
Oct 19 21:52:23 pi4nas systemd[1]: Started Network UPS Tools - power device driver controller.
Oct 19 21:52:23 pi4nas systemd[1]: Stopping Network UPS Tools - power device driver controller...
Oct 19 21:52:23 pi4nas upsdrvctl[17949]: Network UPS Tools - UPS driver controller 2.7.4
Oct 19 21:52:25 pi4nas usbhid-ups[17948]: Signal 15: exiting
Oct 19 21:52:25 pi4nas systemd[1]: nut-driver.service: Succeeded.
Oct 19 21:52:25 pi4nas systemd[1]: Stopped Network UPS Tools - power device driver controller.
```

## Configuración del servidor

El segundo paso es configurar el servidor editando el fichero [`upsd.conf`](https://networkupstools.org/docs/man/upsd.conf.html):

```
sudo vi /etc/nut/upsd.conf
```

En este fichero se pueden especificar bastantes parámetros pero lo mínimo imprescindible es indicar la **dirección IP** y el **puerto** donde escuchará el servidor:

```
LISTEN 127.0.0.1 3493
```

A continuación se definen los **usuarios** que podrán conectarse al servidor y sus **contraseñas** editando el fichero [`upsd.users`](https://networkupstools.org/docs/man/upsd.users.html):

```
sudo vi /etc/nut/upsd.users
```
```
[admin]
password = 7)ycmuIYCbSD=,I0v;
actions = SET
instcmds = ALL

[monitor]
password = N3g<XHkf%HC(Tuw-`n
upsmon master
```

En este ejemplo, se ha creado un usuario que puede realizar todas las operaciones sobre el SAI (`admin`) y otro usuario que únicamente puede monitorizar los parámetros del mismo (`monitor`) con unas contraseñas suficientemente [seguras](https://bitwarden.com/password-generator/).

Finalmente se edita el fichero [`nut.conf`](https://networkupstools.org/docs/man/nut.conf.html) para indicar el modo de funcionamiento **standalone**:

```
sudo vi /etc/nut/nut.conf
```
```
MODE=standalone
```

Una vez realizados los cambios indicados, se puede poner en marcha el `nut-server` mediante el siguiente comando:

```
sudo systemctl start nut-server
```

y comprobar que se inicia correctamente:

```
sudo systemctl status nut-server
```
```
● nut-server.service - Network UPS Tools - power devices information server
   Loaded: loaded (/lib/systemd/system/nut-server.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2022-10-19 21:57:55 CEST; 3s ago
  Process: 18788 ExecStart=/sbin/upsd (code=exited, status=0/SUCCESS)
 Main PID: 18789 (upsd)
    Tasks: 1 (limit: 4915)
   CGroup: /system.slice/nut-server.service
           └─18789 /lib/nut/upsd

Oct 19 21:57:55 pi4nas systemd[1]: Starting Network UPS Tools - power devices information server...
Oct 19 21:57:55 pi4nas upsd[18788]: fopen /var/run/nut/upsd.pid: No such file or directory
Oct 19 21:57:55 pi4nas upsd[18788]: listening on 127.0.0.1 port 3493
Oct 19 21:57:55 pi4nas upsd[18788]: listening on 127.0.0.1 port 3493
Oct 19 21:57:55 pi4nas upsd[18788]: Connected to UPS [mge750]: usbhid-ups-mge750
Oct 19 21:57:55 pi4nas upsd[18788]: Connected to UPS [mge750]: usbhid-ups-mge750
Oct 19 21:57:55 pi4nas upsd[18789]: Startup successful
Oct 19 21:57:55 pi4nas systemd[1]: Started Network UPS Tools - power devices information server.
```

## Comprobación de la comunicación

En este punto es recomendable hacer una primera comprobación usando el comando [`upsc`](https://networkupstools.org/docs/man/upsc.html) para ver si el sistema está funcionando correctamente:

```
sudo upsc mge750@127.0.0.1
```

Si todo está correctamente configurado, el _driver_ se comunicará con el SAI y obtendrá diferentes parámetros del mismo (p.ej. el porcentaje de carga de la batería, si está ONLINE, etc.) tal como se puede ver a continuación:

```
Init SSL without certificate database
battery.charge: 100
battery.charge.low: 30
battery.runtime: 1331
battery.type: PbAc
device.mfr: MGE OPS SYSTEMS
device.model: PROTECTIONCENTER 750
device.serial: 000000000
device.type: ups
driver.name: usbhid-ups
driver.parameter.pollfreq: 30
driver.parameter.pollinterval: 2
driver.parameter.port: auto
driver.parameter.synchronous: no
driver.version: 2.7.4
driver.version.data: MGE HID 1.39
driver.version.internal: 0.41
input.transfer.high: 264
input.transfer.low: 184
outlet.1.desc: PowerShare Outlet 1
outlet.1.id: 2
outlet.1.status: on
outlet.1.switchable: no
outlet.desc: Main Outlet
outlet.id: 1
outlet.switchable: no
output.frequency.nominal: 50
output.voltage: 230.0
output.voltage.nominal: 230
ups.beeper.status: enabled
ups.delay.shutdown: 20
ups.delay.start: 30
ups.load: 12
ups.mfr: MGE OPS SYSTEMS
ups.model: PROTECTIONCENTER 750
ups.power.nominal: 750
ups.productid: ffff
ups.serial: 000000000
ups.status: OL
ups.timer.shutdown: -1
ups.timer.start: -10
ups.vendorid: 0463
```

## Configuración del cliente

Al usar el modo _standalone_, el cliente de `nut` está en el mismo equipo que el servidor y hay que configurarlo editando el fichero [`upsmon.conf`](https://networkupstools.org/docs/man/upsmon.conf.html):

```
sudo vi /etc/nut/upsmon.conf
```

En este fichero se indica la dirección IP del servidor y el SAI que se quiere monitorizar, el usuario que se utilizará (en este caso `monitor`) y su contraseña:

```
MONITOR mge750@127.0.0.1 1 monitor N3g<XHkf%HC(Tuw-`n master
```

A continuación se puede poner en marcha el monitor mediante el siguiente comando:

```
sudo systemctl start nut-monitor
```

y comprobar que todo se inicia correctamente:

```
sudo systemctl status nut-monitor
```
```
● nut-monitor.service - Network UPS Tools - power device monitor and shutdown controller
   Loaded: loaded (/lib/systemd/system/nut-monitor.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2022-10-19 22:03:05 CEST; 2s ago
  Process: 19611 ExecStart=/sbin/upsmon (code=exited, status=0/SUCCESS)
 Main PID: 19613 (upsmon)
    Tasks: 2 (limit: 4915)
   CGroup: /system.slice/nut-monitor.service
           ├─19612 /lib/nut/upsmon
           └─19613 /lib/nut/upsmon

Oct 19 22:03:05 pi4nas systemd[1]: Starting Network UPS Tools - power device monitor and shutdown controller...
Oct 19 22:03:05 pi4nas upsmon[19611]: fopen /var/run/nut/upsmon.pid: No such file or directory
Oct 19 22:03:05 pi4nas upsmon[19611]: UPS: mge750@localhost (master) (power value 1)
Oct 19 22:03:05 pi4nas upsmon[19611]: Using power down flag file /etc/killpower
Oct 19 22:03:05 pi4nas systemd[1]: nut-monitor.service: Can't open PID file /run/nut/upsmon.pid (yet?) after start: No s
Oct 19 22:03:05 pi4nas upsmon[19612]: Startup successful
Oct 19 22:03:05 pi4nas upsmon[19613]: Init SSL without certificate database
Oct 19 22:03:05 pi4nas systemd[1]: nut-monitor.service: Supervising process 19613 which is not our child. We'll most lik
Oct 19 22:03:05 pi4nas systemd[1]: Started Network UPS Tools - power device monitor and shutdown controller.
```

## Monitorización contínua

Se puede utilizar el comando [`upslog`](https://networkupstools.org/docs/man/upslog.html) para realizar una monitorización contínua del SAI de la siguiente manera:

```
sudo upslog -s mge750@127.0.0.1 -l -
```

Por pantalla debería aparecer una línea cada 30 segundos (el intervalo por defecto) indicando algunos parámetros del SAI, como el porcentaje de carga de la batería, su estado (**OL** indica _online_), etc:

```
Network UPS Tools upslog 2.7.4
logging status of mge750@localhost to - (30s intervals)
Init SSL without certificate database
20221019 220747 100 NA 13 [OL] NA NA
20221019 220817 100 NA 13 [OL] NA NA
```

Si se desconecta el SAI de la corriente, se debería observar como disminuye el porcentaje de carga de la batería y cambia el estado del mismo (**OB** indica _out-of-band_):

```
20221019 220917 99 NA 32 [OB] NA NA
20221019 220947 94 NA 34 [OB] NA NA
```

Al conectar de nuevo el cable del SAI, se debería observar que vuelve a cambiar el estado:

```
20221019 221017 90 NA 16 [OL] NA NA
20221019 221047 90 NA 16 [OL] NA NA
```

Además, en la propia consola de la Rasperry Pi se reciben notificaciones _broadcast_ indicando estos cambios:

```
Broadcast message from nut@pi4nas (somewhere) (Wed Oct 19 22:09:05 2022):

UPS mge750@localhost on battery
```
```
Broadcast message from nut@pi4nas (somewhere) (Wed Oct 19 22:10:15 2022):

UPS mge750@localhost on line power
```

Estos mismos mensajes quedan registrados en el fichero de registro habitual en sistemas Linux `/var/log/syslog`:

```
Oct 19 22:09:05 pi4nas upsmon[19613]: UPS mge750@localhost on battery
Oct 19 22:10:15 pi4nas upsmon[19613]: UPS mge750@localhost on line power
```

## Revisión de la seguridad

Aunque la instalación de `nut` mediante paquetes suele tener las protecciones correctas en los ficheros, no está de más comprobar que todos ellos pertenecen a `root:nut` y que tienen la protección `640` tal como se muestra a continuación:

```
sudo ls -l /etc/nut/*
```
```
-rw-r----- 1 root nut  1545 Oct 19 21:57 /etc/nut/nut.conf
-rw-r----- 1 root nut  5626 Oct 19 21:52 /etc/nut/ups.conf
-rw-r----- 1 root nut  4576 Oct 19 21:42 /etc/nut/upsd.conf
-rw-r----- 1 root nut  2243 Oct 19 21:56 /etc/nut/upsd.users
-rw-r----- 1 root nut 15368 Oct 19 22:00 /etc/nut/upsmon.conf
-rw-r----- 1 root nut  3887 Jun  1  2018 /etc/nut/upssched.conf
```

## ¿Y ahora qué?

Esta configuración básica se puede ampliar para [enviar un correo electrónico](https://loganmarchione.com/2017/02/raspberry-pi-ups-monitor-with-nginx-web-monitoring/#email-alerting) cuando el SAI cambie de estado, para monitorizarlo en [_Home Assistant_](https://major.io/2021/03/15/monitor-ups-with-raspberry-pi-zero-w/#adding-homeassistant) con sus gráficas de batería y carga, etc.

Pero ésto se dejará para el siguiente proyecto ;-)

[1]: /assets/img/blog/2022-10-21_image_1.png "MGE Protection Center 675"
[2]: /assets/img/blog/2022-10-21_image_2.png "Esquema de monitorización"
