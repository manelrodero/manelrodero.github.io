---
layout : post
blog-width: true
title: 'Kernel 6.8.12-14-pve provoca 100% en CPU0'
date: '2025-09-13 09:42:04'
#last-updated: '2025-09-13 09:42:04'
published: true
tags:
- Proxmox
author:
  display_name: Manel Rodero
#cover-img: "/assets/img/blog/2025-09-13_cover.png"
thumbnail-img: ""
---

Desde hace unos días, el ventilador del pequeño servidor [Dell Optiplex 7050](mini-pc-dell-optiplex-7050-micro) donde tengo instalado [Proxmox 8.4.13](proxmox-ve-802-en-un-dell-optiplex-7050) se escucha bastante, incluso a cierta distancia fuera del despacho.

## Aumento de la temperatura

Como todos sabemos, la causa principal por la que el ventilador de un ordenador acelera su velocidad es el **aumento de la temperatura interna**. El sistema de refrigeración responde automáticamente cuando detecta que el procesador, la gráfica u otros componentes están alcanzando temperaturas elevadas.

Las principales causas por las que puede subir la temperatura son:

* **Alta carga de trabajo** debido a procesos intensivos en primer o segundo plano
* **Acumulación de polvo** que bloquea el flujo de aire haciendo que el disipador sea menos eficiente
* **Pasta térmica degradada** entre el procesador y el disipador que no permite la transferencia de calor
* **Temperatura ambiente elevada** en verano o habitaciones mal ventiladas
* **Ubicación física del equipo** en superficies blandas o encajado entre muebles
* **Ventilador defectuoso o desalineado** puede hacerlo girar más rápido o hacer ruido mecánico

## Uso excesivo de CPU

En este caso, la causa del aumento de temperatura y el giro más rápido del ventilador es el **uso excesivo de CPU**.

En el **Summary** del servidor Proxmox se puede observar que, aún estando todos los CT y VM parados, el uso de CPU es del 25% de manera sostenida:

![25% of 4 CPU(s)][1]

Para confirmarlo, se instala la herramienta `htop` en el servidor Proxmox y se observa como uno de los núcleos del procesador (`CPU0`) está al 100% contínuamente:

![htop: 100% CPU0][2]

Lo más curioso es que la mayoría de procesos que tienen mayor uso de CPU rondan el 1-4% en la columna `CPU%`.

Entonces, ¿cuáles son las posibles causas de un uso elevado de CPU?

* **Procesos del sistema o _daemons_ atrapados en un bucle** (p.ej. `kworker`)
* **Problemas con el kernel o drivers** en un _hardware_ específico 
* **Sensores o módulos de monitoreo mal configurados** que generan carga innecesaria
* **Problemas con ACPI o gestión de energía** de los estados C/P del procesador

La ejecución de los siguientes comandos no permiten identificar el proceso culpable a primera vista:

```bash
htop
top -o %CPU
ps -eo pid,ppid,cmd,%cpu --sort=-%cpu | head
```

Para activar los **procesos del kernel** en la herramienta `htop` hay que hacer lo siguiente:

* Pulsar `F2` (Setup)
* Seleccionar la categoría _Display options_
* Desactivar la opción `Hide kernel threads`

La idea es intentar ver si aparecen hilos de interrupciones (`irq/xxx`), `ksoftirqd`, `rcu_sched` o algún `kworker` pero tampoco se identifica el culpable.

Finalmente, la ejecución del comando `cat /proc/interrupts` de forma contínua permite ver que hay una línea que crece rápidamente en el núcleo `CPU0`:

```plaintext
root@pve:~# cat /proc/interrupts
            CPU0       CPU1       CPU2       CPU3       
[...]
 NMI:         17         17         14         18   Non-maskable interrupts
 LOC: 130165600 149880862 145607447 135348367 Local timer interrupts
 SPU:          0          0          0          0   Spurious interrupts
 [...]
```

El crecimiento de las **interrupciones locales** en cada núcleo indica que el kernel está generando una cantidad excesiva de interrupciones de temporizador, lo cual puede explicar perfectamente que uno de los núcleos esté al 100% de uso constante.

## Local Timer Interrupts

Son interrupciones periódicas que el kernel utiliza para tareas como:

* Planificación de procesos
* Actualización de contadores de tiempo
* Mantenimiento de estructuras internas

Normalmente, estas ocurren a una frecuencia fija (por ejemplo, 1000 Hz), pero **no deberían saturar un core**.

Si lo hacen, suele deberse a:

* **Bug en el kernel 6.8.12-14-pve** (algunos kernels recientes han tenido problemas con `tick` y `nohz` en CPUs antiguas)
* **Mal funcionamiento del modo `tickless` o `nohz_full`**
* **Problemas con ACPI o firmware EFI**
* **Cargas fantasma o procesos en kernel que no aparecen en `htop`**

Se revisa que no está activado `nohz_full` en la carga de Proxmox:

```plaintext
root@pve:~# cat /proc/cmdline
BOOT_IMAGE=/boot/vmlinuz-6.8.12-14-pve root=/dev/mapper/pve-root ro quiet
```

Se revisa la configuración de los temporizadores y se observa que el kernel tiene activado `CONFIG_TICK_ONESHOT` para reducir los _ticks_ cuando no hay actividad:

```plaintext
root@pve:~# grep -i tick /boot/config-$(uname -r)
CONFIG_TICK_ONESHOT=y
[...]
```

Sin embargo, el hecho de que las **Local Timer Interrupts (LOC)** estén creciendo de forma desmesurada sugiere que el kernel **no está entrando en modo tickless como debería**, o que hay algún componente que está forzando ticks constantes en un core.

## C-states

La hipótesis más probable es que el **kernel 6.8.12-14-pve** podría tener una regresión relacionada con el subsistema de temporizadores o con la gestión de energía en CPUs como el i5-7600T.

Esto puede provocar que **el sistema no entre en estados de reposo profundo (`C-states`)**, y que un core se quede atrapado generando interrupciones de temporizador sin necesidad real.

Los estados de reposo más comunes son:

* **C0 (Estado Activo)**: La CPU está completamente cargada y ejecutando instrucciones activamente
* **C1 (Detención Automática)**: La CPU detiene la ejecución de instrucciones, pero mantiene la coherencia del reloj del núcleo y la caché. Presenta la latencia de activación más rápida de todos los estados C
* **C2 (Detención del Reloj)**: La CPU detiene la señal del reloj, lo que reduce aún más el consumo de energía. Presenta una latencia de activación ligeramente mayor que C1
* **C3 (Suspensión)**: La CPU reduce el voltaje y las señales de reloj, lo que genera un ahorro de energía significativo. La latencia de activación es mayor que en C1 y C2
* **C4 (Suspensión Profunda)**: Este estado implica un voltaje y señales de reloj aún más bajos que en C3, lo que resulta en un mayor ahorro de energía, pero con una mayor latencia
* **C5 (Suspensión Profunda)**: Un estado de suspensión aún más profundo con un consumo de energía mínimo y la latencia de activación más larga
* **C6 (Suspensión Profunda Mejorada)**: Reduce aún más el consumo de energía en comparación con C5. C7 (sueño más profundo): la CPU ingresa a su estado de menor consumo de energía, limpiando los cachés y requiriendo el mayor tiempo de activación

Se instala el paquete `powertop` para obtener información de estos estados ejecutando el comando `powertop --time=10`.

En la pestaña pestaña **Idle stats** se observa como la `CPU0` se mantiene activa en el estado **C0** de forma contínua:

![powertop: Idle states][3]

## Kernel 6.8.12-13-pve

Antes de revisar los parámetros de la BIOS, compruebo que aún dispongo del kernel anterior:

```plaintext
root@pve:~# ls -lh /boot/vmlinuz-*
-rw-r--r-- 1 root root 14M Jul 22 12:00 /boot/vmlinuz-6.8.12-13-pve
-rw-r--r-- 1 root root 14M Aug 27 00:25 /boot/vmlinuz-6.8.12-14-pve
```

Por tanto, reinicio el servidor Proxmox y en la pantalla de **Grub** se selecciona las siguientes opciones para iniciar el kernel anterior:

* Advanced options for Proxmox VE GNU/Linux
* Proxmox VE GNU/Linux, with Linux 6.8.12-13-pve

Después de iniciar el servidor Proxmox con el kernel anterior y con todos los CT y VMs en marcha, **el uso de CPU vuelve a ser bajo** y **el ventilador práctiamente no se escucha**:

![1.51% of 4 CPU(s)][4]

## Microcódigo

Ahora que ya hemos confirmado la existencia de una **incompatibilidad entre el procesador i5-7600T y el kernel 6.8.12-14-pve**, se podrían revisar diferentes cosas:

* **BIOS/UEFI: estados de energía y temporizadores**
  * **Intel SpeedStep / EIST**: activado
  * **C-States**: activado (especialmente C6/C7 si están disponibles)
  * **C1E (Enhanced Halt)**: activado
  * **HPET (High Precision Event Timer)**: activado o desactivado según kernel (puede probarse)
  * **Legacy USB / Wake-on-LAN / PXE boot**: desactivados si no se usan
* **Microcódigo del procesador**
* **Comparar configuración entre kernels**
* **Revisar si hay módulos nuevos o incompatibles**

Lo más sencillo, ahora que el servidor está iniciado, es revisar el **microcódigo** instalado:

```plaintext
root@pve:~# dmesg | grep microcode
[    0.143384] GDS: Vulnerable: No microcode
[    0.843931] microcode: Current revision: 0x000000f0

root@pve:~# cat /proc/cpuinfo | grep microcode
microcode       : 0xf0
microcode       : 0xf0
microcode       : 0xf0
microcode       : 0xf0
```

Como se puede observar, mi CPU es vulnerable a **GDS (Gather Data Sampling)** y no se ha aplicado una mitigación vía microcódigo. Además, la versión cargada seguramente no es la más reciente.

Se puede instalar el microcódigo usando el _helper script_ [Proxmox VE Processor Microcode](https://community-scripts.github.io/ProxmoxVE/scripts?id=microcode&category=Proxmox+%26+Virtualization){:target="_blank"} o, mucho mejor aún, usando el comando `apt`:

```bash
apt update
apt install intel-microcode -y
```

```plaintext
root@pve:~# apt install intel-microcode -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  iucode-tool
The following NEW packages will be installed:
  intel-microcode iucode-tool
0 upgraded, 2 newly installed, 0 to remove and 4 not upgraded.
Need to get 11.2 MB of archives.
After this operation, 19.9 MB of additional disk space will be used.
Get:1 http://ftp.es.debian.org/debian bookworm/main amd64 iucode-tool amd64 2.3.1-3 [56.1 kB]
Get:2 http://ftp.es.debian.org/debian bookworm/non-free-firmware amd64 intel-microcode amd64 3.20250512.1~deb12u1 [11.1 MB]
Fetched 11.2 MB in 0s (41.0 MB/s)     
Selecting previously unselected package iucode-tool.
(Reading database ... 60301 files and directories currently installed.)
Preparing to unpack .../iucode-tool_2.3.1-3_amd64.deb ...
Unpacking iucode-tool (2.3.1-3) ...
Selecting previously unselected package intel-microcode.
Preparing to unpack .../intel-microcode_3.20250512.1~deb12u1_amd64.deb ...
Unpacking intel-microcode (3.20250512.1~deb12u1) ...
Setting up iucode-tool (2.3.1-3) ...
Setting up intel-microcode (3.20250512.1~deb12u1) ...
intel-microcode: microcode will be updated at next boot
Processing triggers for man-db (2.11.2-2) ...
Processing triggers for initramfs-tools (0.142+deb12u3) ...
update-initramfs: Generating /boot/initrd.img-6.8.12-14-pve
Running hook script 'zz-proxmox-boot'..
Re-executing '/etc/kernel/postinst.d/zz-proxmox-boot' in new private mount namespace..
No /etc/kernel/proxmox-boot-uuids found, skipping ESP sync.
```

Como se puede observar en la salida anterior, aunque se instala el microcódigo de forma global en `/lib/firmware/intel-ucode`, únicamente se ha regenerado el kernel actual.

Como quiero probar el microcódigo con el kernel anterior que sabemos que funciona correctamente, es necesario actualizarlo:

```bash
update-initramfs -u -k 6.8.12-13-pve
```

```plaintext
root@pve:~# update-initramfs -u -k 6.8.12-13-pve
update-initramfs: Generating /boot/initrd.img-6.8.12-13-pve
Running hook script 'zz-proxmox-boot'..
Re-executing '/etc/kernel/postinst.d/zz-proxmox-boot' in new private mount namespace..
No /etc/kernel/proxmox-boot-uuids found, skipping ESP sync.
```

A continuación, se reinicia el servidor Proxmox y se vuelve a seleccionar el kernel anterior.

Se comprueba que la **revisión 0xf8 del microcódigo de Intel** se ha instalado correctamente:

```plaintext
root@pve:~# dmesg | grep microcode
[    0.870131] microcode: Current revision: 0x000000f8
[    0.870133] microcode: Updated early from: 0x000000f0

root@pve:~# cat /proc/cpuinfo | grep microcode
microcode       : 0xf8
microcode       : 0xf8
microcode       : 0xf8
microcode       : 0xf8
```

El uso de CPU y velocidad de giro del ventilador siguen siendo bajos tal como muestra esta captura:

![microcode 0xf8][5]

## Kernel 6.8.12-14-pve

Ahora toca la "prueba de fuego", reiniciar el servidor Proxmox y dejar que cargue el último kernel `6.8.12-14-pve` para comprobar si el problema era debido a cambios en la gestión de energía que dependiesen del microcódigo.

¡Bingo! Hemos tenido suerte y las correcciones incluidas en el microcódigo `0xf8` son importantes para que:

* El kernel pueda aplicar ciertas mitigaciones
* El procesador entre en estados de reposo (**C1**, **C6**, etc.)
* La `CPU0` no quede atrapada en **C0**, generando carga fantasma

![kernel 6.8.12-14-pve][6]

¡Qué tranquilidad y qué silencio! ;-)

### Historial de cambios

* **2025-09-13**: Documento inicial

[1]: /assets/img/blog/2025-09-13_image_1.png "25% of 4 CPU(s)"
[2]: /assets/img/blog/2025-09-13_image_2.png "htop: 100% CPU0"
[3]: /assets/img/blog/2025-09-13_image_3.png "powertop: Idle states"
[4]: /assets/img/blog/2025-09-13_image_4.png "1.51% of 4 CPU(s)"
[5]: /assets/img/blog/2025-09-13_image_5.png "microcode 0xf8"
[6]: /assets/img/blog/2025-09-13_image_6.png "kernel 6.8.12-14-pve"
