---
layout : post
blog-width: true
title: 'Instalar Pi-hole en Proxmox LXC'
date: '2023-11-18 18:30:15'
last-updated: '2025-02-13 19:25:40'
published: true
tags:
- Proxmox
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2023-11-18_cover.png"
thumbnail-img: ""
---

Para seguir el proceso de migración de mi _Home Lab_ desde una pequeña [Raspberry Pi 4](instalar-raspberry-pi-os-64bits) a un [OptiPlex 7050 ejecutando Proxmox](proxmox-ve-802-en-un-dell-optiplex-7050) ahora le toca el turno a [**Pi-hole**](https://pi-hole.net/){:target="_blank"}, un sumidero de DNS que protege a los dispositivos de contenido no deseado, sin instalar ningún programa del lado del cliente.

En esta ocasion, en lugar de realizar la [instalación de Pi-hole en Docker](instalacion-de-pihole-en-docker), se utilizará un contenedor ligero LXC de Proxmox con TurnKey Core. El procedimiento para crear este LXC es el mismo que usé para [ejecutar Docker en Proxmox LXC](docker-en-proxmox-lxc-con-turnkey-core).

# Características del LXC

Este **sumidero de DNS** con [Pi-hole](https://pi-hole.net/){:target="_blank"} se ejecutará sobre un LXC con las siguientes características:

* LXC no privilegiado, sin _nesting_ y sin `keyctl`
* CT ID 304
* Password + SSH key file
* Disco de **8GB**
* CPU con 1 _core_
* 512MB de memoria
* IP fija 192.168.1.81

```
pct create 304 local:vztmpl/debian-12-turnkey-core_18.0-1_amd64.tar.gz \
  --ostype debian --arch amd64 \
  --hostname pihole1 --unprivileged 1 --password P@ssw0rd --ssh-public-keys /root/id_edcsa.pub \
  --storage local-lvm --rootfs volume=local-lvm:8 \
  --cores 1 \
  --memory 512 --swap 512 \
  --net0 name=eth0,bridge=vmbr0,firewall=1,ip=192.168.1.81/24,gw=192.168.1.1 \
  --start false
```

A continuación se pone en marcha el LXC desde la CLI de Proxmox y se procede a configurarlo de la misma manera que ya expliqué en el artículo sobre [LXC y Docker](docker-en-proxmox-lxc-con-turnkey-core):

```
pct start 304
```

# Acceder al LXC

Se puede acceder al LXC utilizando la opción `Console` de la GUI o, mucho mejor, mediante **SSH** gracias a la configuración de las claves públicas que se hizo en el mismo:

```Bash
ssh root@192.168.1.81
```

Una vez dentro del LXC, hay que: 

* completar el asistente de instalación
* reiniciar el LXC (sobretodo si hay una actualización del _kernel_)
* reconfigurar la zona horaria (`dpkg-reconfigure tzdata` y seleccionar `Europe/Madrid`)
* actualizar los paquetes (`apt update` y `apt upgrade -y`)
* desinstalar el paquete `etckeeper` (`apt purge etckeeper`)
* reiniciar el LXC

# Instalación de Pi-hole

Antes de comenzar la instalación es necesario [deshabilitar el `DNSStubListener`](https://github.com/pi-hole/docker-pi-hole#installing-on-ubuntu-or-fedora){:target="_blank"} de forma manual para evitar errores al reconfigurar el servicio `systemd-resolved`:

```
sed -r -i.orig 's/#?DNSStubListener=yes/DNSStubListener=no/g' /etc/systemd/resolved.conf
systemctl restart systemd-resolved.service
```

{: .box-note}
En TurnKey Core 18.0 (basado en Debian 12) no existe el fichero anterior y el servicio está _masked_.

La [instalación de Pi-hole](https://docs.pi-hole.net/main/basic-install/){:target="_blank"} en una distribución basada en Debian es muy sencilla usando el _script_ `basic-install.sh`:

```
cd ~
wget -O basic-install.sh https://install.pi-hole.net
bash basic-install.sh
```

Si se cumplen los requisitos para instalar Pi-hole, se ejecutará el asistente _Pi-hole Automated Installer_ mediante el cual se proporcionará la información necesaria para la instalación:

* Static IP Needed: **Continue**
* Upstream DNS Provider: `Quad9 (filtered, DNSSEC)`
* Blocklist: **Yes** (para incluir la lista _StevenBlack's Unified Host List_)
* Admin Web Interface: **Yes**
* Web Server: **Yes**
* Enable Logging: **Yes**
* Privacy Mode: `Show everything`

Al finalizar la instalación, si no hay ningún problema, aparecerá una pantalla mostrando el **password** generado de forma aleatoria y la URL de la interfaz web de administración [http://192.168.1.81/admin](http://192.168.1.81/admin).

{: .box-note}
**Nota**: Se puede utilizar el comando `pihole -a -p` para cambiar la constraseña de la interfaz web y el comando `pihole -up` para actualizar la versión.

# Forwarding vs Recursive

En una instalación normal de Pi-hole, se utiliza una configuración **Forwarding** para reenviar al siguiente proveedor de DNS aquellas peticiones que no pueda resolver de manera local consultando la lista de bloqueos.

Esta configuración, aunque es funcional y sencilla, tiene algunos problemas de privacidad y seguridad:

* **Privacidad**: El proveedor de DNS podría tener un listado completo de las páginas web que visitamos si guarda información sobre los dominios que pide un determinado cliente.
* **Seguridad**: Podríamos ser víctimas de un ataque de [_DNS Spoofing_](https://powerdmarc.com/what-is-dns-spoofing/){:target="_blank"} que nos redirija a una página _fake_ si el proveedor DNS altera las respuestas.

Por este motivo es recomendable utilizar una configuración **Recursive** mediante [`unbound`](https://docs.pi-hole.net/guides/dns/unbound/){:target="_blank"} donde, en lugar de preguntar a un proveedor de DNS, se consulta la IP directamente al servidor autoritativo del dominio que se está buscando.

{: .box-note}
**Nota**: Otra opción, sería utilizar un **DNS-over-HTTPS** mediante [`cloudflared`](https://docs.pi-hole.net/guides/dns/cloudflared/){:target="_blank"} para evitar las peticiones en texto plano tal como hace [mi compañero Roberto P. Rubio](https://robertoprubio.github.io/instalacion-de-pihole-y-cloudflared-doh/){:target="_blank"}.

# Instalación de `unbound`

La instalación de `unbound` se realiza de forma muy sencilla desde los repositorios usando el comando `apt`:

```
apt install unbound -y
```

{: .box-note}
**Nota**: Inicialmente puede que aparezca el error `Failed to start Unbound DNS server` ya que el puerto **53** está siendo usado por Pi-hole. 

## Configuración

Es necesario crear un fichero de configuración de `unbound` para indicar, entre otras cosas, el puerto **5353** en el que tiene que escuchar para que no haya conflictos:

```
nano /etc/unbound/unbound.conf.d/pi-hole.conf
```

Escribir el siguiente contenido y grabar el fichero de configuración:

```
server:
    # If no logfile is specified, syslog is used
    # logfile: "/var/log/unbound/unbound.log"
    verbosity: 0

    interface: 127.0.0.1
    port: 5353
    do-ip4: yes
    do-udp: yes
    do-tcp: yes

    # May be set to yes if you have IPv6 connectivity
    do-ip6: no

    # You want to leave this to no unless you have *native* IPv6. With 6to4 and
    # Terredo tunnels your web browser should favor IPv4 for the same reasons
    prefer-ip6: no

    # Use this only when you downloaded the list of primary root servers!
    # If you use the default dns-root-data package, unbound will find it automatically
    #root-hints: "/var/lib/unbound/root.hints"

    # Trust glue only if it is within the server's authority
    harden-glue: yes

    # Require DNSSEC data for trust-anchored zones, if such data is absent, the zone becomes BOGUS
    harden-dnssec-stripped: yes

    # Don't use Capitalization randomization as it known to cause DNSSEC issues sometimes
    # see https://discourse.pi-hole.net/t/unbound-stubby-or-dnscrypt-proxy/9378 for further details
    use-caps-for-id: no

    # Reduce EDNS reassembly buffer size.
    # IP fragmentation is unreliable on the Internet today, and can cause
    # transmission failures when large DNS messages are sent via UDP. Even
    # when fragmentation does work, it may not be secure; it is theoretically
    # possible to spoof parts of a fragmented DNS message, without easy
    # detection at the receiving end. Recently, there was an excellent study
    # >>> Defragmenting DNS - Determining the optimal maximum UDP response size for DNS <<<
    # by Axel Koolhaas, and Tjeerd Slokker (https://indico.dns-oarc.net/event/36/contributions/776/)
    # in collaboration with NLnet Labs explored DNS using real world data from the
    # the RIPE Atlas probes and the researchers suggested different values for
    # IPv4 and IPv6 and in different scenarios. They advise that servers should
    # be configured to limit DNS messages sent over UDP to a size that will not
    # trigger fragmentation on typical network links. DNS servers can switch
    # from UDP to TCP when a DNS response is too big to fit in this limited
    # buffer size. This value has also been suggested in DNS Flag Day 2020.
    edns-buffer-size: 1232

    # Perform prefetching of close to expired message cache entries
    # This only applies to domains that have been frequently queried
    prefetch: yes

    # One thread should be sufficient, can be increased on beefy machines.
    # In reality for most users running on small networks or on a single machine, it should
    # be unnecessary to seek performance enhancement by increasing num-threads above 1.
    num-threads: 1

    # Ensure kernel buffer is large enough to not lose messages in traffic spikes
    so-rcvbuf: 1m

    # Ensure privacy of local IP ranges
    private-address: 192.168.0.0/16
    private-address: 169.254.0.0/16
    private-address: 172.16.0.0/12
    private-address: 10.0.0.0/8
    private-address: fd00::/8
    private-address: fe80::/10
```

También es recomendable añadir la siguiente línea al fichero `/etc/dnsmasq.d/99-edns.conf` para limitar el tamaño de paquete que utilizará Pi-hole FTL tal como se indicó en el [_DNS Flag Day 2020_](https://blog.apnic.net/2020/09/17/dns-flag-day-2020-what-you-need-to-know/){:target="_blank"}:

```
edns-packet-max=1232
```

A continuación se reinicia el servicio con la nueva configuración y, si todo está bien, debería iniciar correctamente:

```
systemctl restart unbound.service 
systemctl status unbound.service
```

## Comprobación de resolución

Para comprobar que `unbound` resuelve nombres correctamente, se puede ejecutar el comando `dig` indicando la dirección IP y puerto donde escucha:

```
dig pi-hole.net @127.0.0.1 -p 5353
```

Si funciona correctamente, se debería obtener un registro de tipo `A` tal como se muestra a continuación:

```
; <<>> DiG 9.16.44-Debian <<>> pi-hole.net @127.0.0.1 -p 5353
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 54721
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;pi-hole.net.                   IN      A

;; ANSWER SECTION:
pi-hole.net.            300     IN      A       3.18.136.52

;; Query time: 71 msec
;; SERVER: 127.0.0.1#5353(127.0.0.1)
;; WHEN: Sat Nov 18 21:22:21 CET 2023
;; MSG SIZE  rcvd: 56
```

También se puede realizar una prueba usando [**DNSSEC**](https://www.cloudflare.com/dns/dnssec/how-dnssec-works/){:target="_blank"} para resolver el dominio [`dnssec.works`](https://dnssec.works/){:target="_blank"}:

```
dig dnssec.works +dnssec +multi @127.0.0.1 -p 5353
```
```
; <<>> DiG 9.16.44-Debian <<>> dnssec.works +dnssec +multi @127.0.0.1 -p 5353
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 52167
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags: do; udp: 1232
;; QUESTION SECTION:
;dnssec.works.          IN A

;; ANSWER SECTION:
dnssec.works.           3514 IN A 5.45.107.88
dnssec.works.           3514 IN RRSIG A 8 2 3600 (
                                20231211004428 20231110235251 63306 dnssec.works.
                                4AWfdRz9vpEsFEyvy11sh+wOBaTGFPUBDjND/RUu1Er4
                                +P0Pua9ieH2QAu/3Q9XkU/q9fL6KqXHMzP4/6MjKkXpb
                                SFyliuk6wHVuKsC5Ac54IN/bf85TET4cotY+Nufjh0ww
                                ucOgVUh7sJVbQjjmpE0cryyipWU/2mib8Nbk0hp9fx0U
                                d/0H52QxiR24yRvtuXZbaQTZvSzwPc+vIozsqegBOpQW
                                3dbQQCej8Dn808E9L6mZPmajUUdN+GXPyfZ3 )

;; Query time: 0 msec
;; SERVER: 127.0.0.1#5353(127.0.0.1)
;; WHEN: Sat Nov 18 21:27:17 CET 2023
;; MSG SIZE  rcvd: 293
```

## Uso en Pi-hole

Para hacer que Pi-hole utilice `unbound` en lugar de los servidores indicados durante la instalación, hay que hacer lo siguiente:

* Iniciar sesió en la interfaz web
* Acceder a la sección `Settings`
* Acceder a la pestaña `DNS`
* Desmarcar los _Upstream DNS Servers_ de Quad9
* Añadir uno de tipo _custom_: **127.0.0.1#5353**
* Grabar la configuración

{: .box-note}
**Nota**: Con la configuración de `unbound` indicada anteriormente, los nombres que apunten a direcciones privadas según el [**RFC1918**](https://datatracker.ietf.org/doc/html/rfc1918.html){:target="_blank"} (por ejemplo `192.168.1.180`) no se resuelven por motivos de seguridad.<br><br>Si este es nuestro caso, se tiene que agregar la línea [`private-domain`](https://unbound.docs.nlnetlabs.nl/en/latest/manpages/unbound.conf.html#unbound-conf-private-domain){:target="_blank"} con el nombre del dominio que contiene direcciones privadas al fichero de configuración de `unbound`.

```
    # Resolver mi dominio
    private-domain: midominio.com
```

# Servidor secundario

Para evitar interrupciones en la resolución de nombres si el servicio Pi-hole no está disponible (por ejemplo cuando se actualiza o si tiene algún problema) es recomendable tener un segundo servidor.

Aunque lo ideal sería tenerlo en una máquina física diferente, en este caso se aprovechará la flexibilidad de Proxmox para **clonar** el contenedor LXC en su estado actual.

Antes de ponerlo en marcha habrá que cambiar el nombre y la dirección IP del mismo para evitar conflictos:

* Network &rarr; IP Address: `192.168.1.82/24`
* DNS &rarr; Hostname: `pihole22`

{: .box-note}
**Nota**: Al clonar el primer contenedor LXC, éste tendrá las mismas claves SSH del servidor lo cual provoca confusión a la hora de conectarse (`This host key is known by the following other names/addresses`).<br><br>Para solucionarlo hay que conectarse a la `Console` del LXC en Proxmox para borrar las claves actuales **`rm /etc/ssh/ssh_host_*`** y reconfigurar el servidor SSH **`dpkg-reconfigure openssh-server`**.

# Referencias

* [You're running Pi-Hole wrong! Setting up your own Recursive DNS Server!](https://youtu.be/FnFtWsZ8IP0){:target="_blank"}, vídeo del canal "[Craft Computing](https://www.youtube.com/@CraftComputing){:target="_blank"}"
* [Pi-hole as All-Around DNS Solution using Unbound](https://docs.pi-hole.net/guides/dns/unbound/){:target="_blank"}
* [Installing Pi-Hole inside a Proxmox LXC Container](https://www.datahoards.com/installing-pi-hole-inside-a-proxmox-lxc-container/){:target="_blank"}
* [Fixing Failed Network Name Resolution in Ubuntu with systemd-resolved](https://devicetests.com/fixing-network-name-resolution-ubuntu){:target="_blank"}
* [How to Resolve Port 53 Conflict Between, systemd.resolved and pihole-dnscrypt docker](https://raspberrypi.stackexchange.com/questions/128288/how-to-resolve-port-53-conflict-between-systemd-resolved-and-pihole-dnscrypt-do){:target="_blank"}
* [PiHole Unbound vs Cloudflared-DOH](https://robertoprubio.github.io/pi-hole-unbound-vs-cloudflare-doh/){:target="_blank"}, artículo de Roberto P. Rubio
* [Unbound not resolving some domains](https://stackoverflow.com/questions/68210563/unbound-not-resolving-some-domains){:target="_blank"}
* [12 Dig Command Examples To Query DNS In Linux](https://www.rootusers.com/12-dig-command-examples-to-query-dns-in-linux/){:target="_blank"}
* [DNSSEC Resolver Test](https://wander.science/projects/dns/dnssec-resolver-test/){:target="_blank"}

### Historial de cambios

* **2023-11-18**: Documento inicial
* **2024-06-22**: Revisión y creación de LXC mediante CLI
* **2025-02-15**: Añadir comandos iniciales
