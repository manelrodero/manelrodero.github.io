---
layout : post
blog-width: true
title: 'Instalación de Network UPS Tools (NUT) en Proxmox'
date: '2023-11-26 09:01:38'
published: true
tags:
- Proxmox
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2023-11-26_cover.png"
thumbnail-img: ""
---

Para seguir el proceso de migración de mi _Home Lab_ desde una pequeña [Raspberry Pi 4](instalar-raspberry-pi-os-64bits) a un [OptiPlex 7050 ejecutando Proxmox](proxmox-ve-802-en-un-dell-optiplex-7050) ahora le toca el turno a [**Network UPS Tool (NUT)**](https://networkupstools.org/){:target="_blank"}, un proyecto que proporciona un protocolo común para monitorizar y administrar dispositivos de energía.

La instalación de NUT se realizará en el **_host_ de Proxmox** mediante un procedimiento similar al seguido para [monitorizar un SAI (UPS) desde una Raspberry Pi](monitorizar-un-sai-ups-desde-una-raspberry-pi).

La principal ventaja de instalarlo así, es que no hace falta instalar el cliente de NUT en los contenedores LXC o en las VM ya que el propio _host_ de Proxmox se encargará de apagarlos correctamente cuando sea necesario.

En esta instalación utilizaremos la [configuración avanzada de NUT](https://networkupstools.org/docs/user-manual.chunked/ar01s03.html#_monitoring_diagrams){:target="_blank"}, también llamada **netserver**, donde el _driver_ para comunicarse con el SAI (UPS) y los servicios `upsd` y `upsmon` se ejecutan en el _host_ de Proxmox. Al mismo tiempo, esta configuración permite que otras máquinas se conecten al servidor NUT a través de la red.

![Overview of NUT][1]

# Network UPS Tools

## Detección del SAI

Para poder realizar la monitorización del SAI es necesario que el sistema operativo lo detecte correctamente. Después de conectar el cable USB del SAI a un puerto USB del _host_, se ejecuta el comando `lsusb` para listar los dispositivos USB reconocidos.

Si todo funciona correctamente, debería aparecer una línea correspondiente al SAI (en mi caso un [MGE Protection Center 750](http://powerquality.eaton.com/about-us/mgeops.asp){:target="_blank"} identificado como `MGE UPS Systems UPS`):

```
Bus 001 Device 003: ID 0463:ffff MGE UPS Systems UPS
```

Si usamos las opciones `-v -s 1:3` se puede obtener información más detallada de este dispositivo y se puede comprobar que soporta la [interfaz USB/HID](https://networkupstools.org/docs/man/usbhid-ups.html){:target="_blank"} (_Human Interface Device_):

```
root@pve:~# lsusb -v -s 1:3

Bus 001 Device 003: ID 0463:ffff MGE UPS Systems UPS
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               1.10
  bDeviceClass            0 
  bDeviceSubClass         0 
  bDeviceProtocol         0 
  bMaxPacketSize0         8
  idVendor           0x0463 MGE UPS Systems
  idProduct          0xffff UPS
  bcdDevice           42.41
  iManufacturer           1 MGE OPS SYSTEMS
  iProduct                2 PROTECTIONCENTER
  iSerial                 4 000000000
  bNumConfigurations      1
  Configuration Descriptor:
    bLength                 9
    bDescriptorType         2
    wTotalLength       0x0022
    bNumInterfaces          1
    bConfigurationValue     1
    iConfiguration          0 
    bmAttributes         0xa0
      (Bus Powered)
      Remote Wakeup
    MaxPower               20mA
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        0
      bAlternateSetting       0
      bNumEndpoints           1
      bInterfaceClass         3 Human Interface Device
      bInterfaceSubClass      0 
      bInterfaceProtocol      0 
      iInterface              0 
        HID Device Descriptor:
          bLength                 9
          bDescriptorType        33
          bcdHID               1.00
          bCountryCode           33 US
          bNumDescriptors         1
          bDescriptorType        34 Report
          wDescriptorLength     769
         Report Descriptors: 
           ** UNAVAILABLE **
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x81  EP 1 IN
        bmAttributes            3
          Transfer Type            Interrupt
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0008  1x 8 bytes
        bInterval              20
Device Status:     0x0001
  Self Powered
```

## Instalación de NUT

Aunque lo más sencillo sería instalar el _metapackage_ `nut` desde los repositorios oficiales usando el comando `apt`, en este caso optaremos por instalar únicamente el **servidor** y el **cliente** de NUT:

```
apt update && apt install nut-server nut-client -y
```

La siguiente tabla muestra los diferentes paquetes que forman parte de NUT:

| **Paquete** | **Descripción** |
|---|---|
| nut | network UPS tools - metapackage |
| nut-cgi | network UPS tools - web interface |
| **nut-client** | network UPS tools - clients |
| nut-doc | network UPS tools - documentation |
| nut-i2c | network UPS tools - I2C driver |
| nut-ipmi | network UPS tools - IPMI driver |
| nut-modbus | network UPS tools - Modbus driver |
| nut-monitor | network UPS tools - GUI application to monitor UPS status |
| nut-powerman-pdu | network UPS tools - PowerMan PDU driver |
| **nut-server** | network UPS tools - core system |
| nut-snmp | network UPS tools - SNMP driver |
| nut-xml | network UPS tools - XML/HTTP driver |

## Copia de seguridad inicial

Antes de comenzar con la configuración inicial de NUT para este entorno, se procede a realizar una copia de seguridad de los ficheros de configuración originales:

```
mkdir /etc/nut/original
chown root:nut /etc/nut/original
chmod 750 /etc/nut/original
cp /etc/nut/*.conf /etc/nut/original/
cp /etc/nut/*.users /etc/nut/original/
```

## Configuración de NUT

### Búsqueda de dispositivos USB

El primer paso para configurar NUT consiste en **buscar los dispositivos USB compatibles** usando el comando `nut-scanner -U`:

```
root@pve:~# nut-scanner -U
Scanning USB bus.
[nutdev1]
        driver = "usbhid-ups"
        port = "auto"
        vendorid = "0463"
        productid = "FFFF"
        product = "PROTECTIONCENTER"
        serial = "000000000"
        vendor = "MGE OPS SYSTEMS"
        bus = "001"
```

### Modo del servidor (`/etc/nut/nut.conf`)

El modo de funcionamiento del servidor, en este caso **netserver**, se configura en el fichero [`/etc/nut/nut.conf`](https://networkupstools.org/docs/man/nut.conf.html){:target="_blank"}:

```
MODE=netserver
```

### Driver de dispositivo (`/etc/nut/ups.conf`)

Para que NUT se pueda comunicarse con el SAI necesita un _driver_ específico para cada [marca y modelo de la HCL](https://networkupstools.org/stable-hcl.html){:target="_blank"}. En este caso, se utilizará el _driver_ [**USB/HID**](https://networkupstools.org/docs/man/usbhid-ups.html){:target="_blank"} que permite detectar y usar cualquier SAI que utilice la clase _HID Power Device Class_. La cantidad de datos proporcionada variará en función de la marca y modelo.

La configuración del SAI se añadirá al final del fichero [`/etc/nut/ups.conf`](https://networkupstools.org/docs/man/ups.conf.html){:target="_blank"} usando los valores obtenidos anteriormente:

```
[mge750]
# MGE Protection Center 750
  driver = usbhid-ups
  port = auto
  desc = "MGE Protection Center 750"
  vendorid = 0463
  productid = FFFF
  serial = 000000000
```

A continuación se ejecuta el comando `upsdrvctl start` y, si la configuración es correcta, debería mostrarse algo similar a:

```
root@pve:/etc/nut# upsdrvctl start
Network UPS Tools - UPS driver controller 2.8.0
Network UPS Tools - Generic HID driver 0.47 (2.8.0)
USB communication driver (libusb 1.0) 0.43
Using subdriver: MGE HID 1.46
```

{: .box-note}
**Nota**: La documentación oficial indica que es mejor utilizar el comando [`upsdrvsvcctl`](https://networkupstools.org/docs/man/upsdrvsvcctl.html){:target="_blank"}, un _wrapper_ para controlar los _drivers_ usando servicios. Por ejemplo, este _script_ permite reconfigurar los servicios después de una actualización del paquete NUT.

El comando `systemctl` permite obtener información sobre el _driver_ `nut-driver@mge750.service`:

```
root@pve:~# systemctl status nut-driver@mge750.service 
● nut-driver@mge750.service - Network UPS Tools - device driver for NUT device 'mge750'
     Loaded: loaded (/lib/systemd/system/nut-driver@.service; enabled; preset: enabled)
    Drop-In: /etc/systemd/system/nut-driver@mge750.service.d
             └─nut-driver-enumerator-generated-checksum.conf, nut-driver-enumerator-generated.conf
     Active: active (running) since Sun 2023-11-26 11:21:15 CET; 39min ago
    Process: 24077 ExecStart=/bin/sh -c NUTDEV="`/usr/libexec/nut-driver-enumerator.sh --get-device-for-service mge750`" \
    && [ -n "$NUTDEV" ] || { echo >
   Main PID: 24192 (usbhid-ups)
      Tasks: 1 (limit: 18781)
     Memory: 588.0K
        CPU: 418ms
     CGroup: /system.slice/system-nut\x2ddriver.slice/nut-driver@mge750.service
             └─24192 /lib/nut/usbhid-ups -a mge750
```

## Fichero `/etc/nut/upsd.conf`

En el fichero [`/etc/nut/upsd.conf`](https://networkupstools.org/docs/man/upsd.conf.html){:target="_blank"} se pueden especificar bastantes parámetros pero lo mínimo imprescindible es indicar la **dirección IP** y el **puerto** donde escuchará el servidor:

```
LISTEN 127.0.0.1 3493
```

## Fichero `/etc/nut/upsd.users`

Los usuarios que podrán conectarse al servidor, sus permisos y sus contraseñas se definen en el fichero [`/etc/nut/upsd.users`](https://networkupstools.org/docs/man/upsd.users.html){:target="_blank"}:

```
[admin]
password = 7yycmuIYCbSDKnI0vB
actions = SET
actions = FSD
instcmds = ALL
upsmon master

[monitor]
password = N3gbXHkfzHCxTuwABn
upsmon slave
```

En este ejemplo, se ha creado un usuario que puede realizar todas las operaciones sobre el SAI (`admin`) y otro usuario que únicamente puede monitorizar los parámetros del mismo (`monitor`) con unas contraseñas suficientemente seguras.

## Comprobación de comunicación

A continuación se puede iniciar el servicio `nut-server` y comprobar su estado mediante el comando `systemctl`:

```
root@pve:~# systemctl start nut-server.service
root@pve:~# systemctl status nut-server.service 
● nut-server.service - Network UPS Tools - power devices information server
     Loaded: loaded (/lib/systemd/system/nut-server.service; enabled; preset: enabled)
     Active: active (running) since Sun 2023-11-26 11:59:23 CET; 3min 40s ago
   Main PID: 39574 (upsd)
      Tasks: 1 (limit: 18781)
     Memory: 624.0K
        CPU: 9ms
     CGroup: /system.slice/nut-server.service
             └─39574 /lib/nut/upsd -F
```

Finalmente comprobamos que existe comunicación desde un cliente mediante el comando `upsc`:

```
upsc mge750@127.0.0.1
```

Si todo está correctamente configurado, el driver se comunicará con el SAI y obtendrá diferentes parámetros del mismo (p.ej. el porcentaje de carga de la batería, si está ONLINE, etc.) tal como se puede ver a continuación:

```
root@pve:~# upsc mge750@127.0.0.1
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
driver.parameter.productid: FFFF
driver.parameter.serial: 000000000
driver.parameter.synchronous: auto
driver.parameter.vendorid: 0463
driver.version: 2.8.0
driver.version.data: MGE HID 1.46
driver.version.internal: 0.47
driver.version.usb: libusb-1.0.26 (API: 0x1000109)
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
ups.beeper.status: disabled
ups.delay.shutdown: 20
ups.delay.start: 30
ups.load: 15
ups.mfr: MGE OPS SYSTEMS
ups.model: PROTECTIONCENTER 750
ups.power.nominal: 750
ups.productid: ffff
ups.realpower: 90
ups.serial: 000000000
ups.status: OL
ups.timer.shutdown: -1
ups.timer.start: -10
ups.vendorid: 0463
```

## Fichero `/etc/nut/upsmon.conf`

Los parámetros de monitorización se definen en el fichero [`/etc/nut/upsmon.conf`](https://networkupstools.org/docs/man/upsmon.conf.html){:target="_blank"}.

En este fichero se indica la dirección IP del servidor y el SAI que se quiere monitorizar, el usuario que se utilizará (en este caso `monitor`) y su contraseña:

```
MONITOR mge750@127.0.0.1 1 monitor N3gbXHkfzHCxTuwABn master
```

# Referencias

* [NUT Configuration Examples](https://github.com/networkupstools/ConfigExamples/releases/latest){:target="_blank"}, libro en PDF mantenido por Roger Price
* [DIY HOME SERVER 2021 – Software – PROXMOX – NUT UPS Monitoring](https://www.kreaweb.be/diy-home-server-2021-software-proxmox-ups/){:target="_blank"}
* [Administración básica del sai Salicru SPSOne 900VA con NUT](https://www.jormc.es/tutoriales/administracion-basica-del-sai-salicru-spsone-900va-con-nut/){:target="_blank"}
* [Set up & Monitor your UPS: Proxmox & Home-Assistant](https://www.thesmarthomebook.com/2022/09/02/setting-up-monitor-your-ups-proxmox-home-assistant/){:target="_blank"}
* [UPS Nut Configuration Tutorial](https://www.sobyte.net/post/2023-03/ups-nut-configuration-tutorial/){:target="_blank"}

[1]: /assets/img/blog/2023-11-26_image_1.png "Overview of NUT"
