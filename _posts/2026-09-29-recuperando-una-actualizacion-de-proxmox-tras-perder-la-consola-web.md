---
layout : post
blog-width: true
title: 'Recuperando una actualización de Proxmox tras perder la consola web'
date: '2026-09-29 09:24:22'
#last-updated: '2026-09-29 09:24:22'
published: true
tags:
- Proxmox
author:
  display_name: Manel Rodero
#cover-img: "/assets/img/blog/2026-09-29_cover.png"
thumbnail-img: ""
---

# Recuperando una actualización de Proxmox tras perder la consola web

Durante una actualización de Proxmox ejecutada desde la consola **Shell** de la interfaz web se produjo un corte de red que la dejó "congelada".

Al recargar la página web para intentar recuperarla, apareció una nueva shell y se perdió la sesión original donde se estaba ejecutando el comando `pveupgrade`.

Al ejecutar de nuevo `pveupgrade` para intentar continuar la actualización apareció el siguiente mensaje de error:

```text
Starting system upgrade: apt-get dist-upgrade
E: Could not get lock /var/lib/dpkg/lock-frontend.
It is held by process 1947719 (apt-get)
```

Y al intentar reiniciar con el comando `reboot` (_spoiler: no se debería haber intentado_) apareció el siguiente mensaje de error:

```text
Operation inhibited by "APT"
```

Esto indicaba que la actualización original no había terminado y que seguía existiendo un proceso `apt-get` activo.

## Identificación del proceso bloqueante

APT indicó directamente qué proceso mantenía el bloqueo:

```text
It is held by process 1947719 (apt-get)
```

A partir de ahí se verificó su estado:

```bash
ps -fp 1947719
```

Resultado:

```text
UID      PID      PPID  CMD
root  1947719 1947718   apt-get dist-upgrade
```

Por tanto, la actualización seguía viva.

## Diagnóstico

El siguiente paso fue inspeccionar el árbol de procesos:

```bash
pstree -p 1947719
```

Resultado:

```text
apt-get
 └─ sh
     └─ apt-listchanges
         └─ sensible-pager
             └─ pager
```

Esta salida fue la **clave**.

La actualización todavía no estaba instalando paquetes. Se encontraba detenida dentro de `apt-listchanges`, una utilidad que muestra cambios y notas importantes de los paquetes antes de continuar con la instalación.

La confirmación llegó mediante:

```bash
strace -p 1947719
```

que mostraba:

```text
wait4(...)
```

Es decir, `apt-get` estaba esperando a que finalizara uno de sus procesos hijos.

## ¿Qué había ocurrido realmente?

Justo antes del corte de red, `apt-listchanges` estaba mostrando información mediante un paginador similar a `less`.

En una situación normal el usuario debe:

- avanzar con la barra espaciadora,
- o salir pulsando la tecla `q`.

Al perderse la consola web, el paginador quedó asociado a un terminal que ya no existía para el usuario.

Como consecuencia:

- `apt-listchanges` quedó esperando una entrada de teclado,
- `apt-get` quedó esperando a `apt-listchanges`,
- `dpkg` todavía no había comenzado a instalar paquetes.

La actualización no estaba rota ni a medias.

Simplemente estaba pausada esperando una interacción imposible de realizar.

## Solución

Se eliminó únicamente el paginador:

```bash
kill 1949095
```

Inmediatamente después:

```bash
pstree -p 1947719
```

pasó a mostrar:

```text
apt-get
 └─ dpkg
```

y posteriormente:

```text
apt-get
 └─ dpkg
     └─ dpkg-deb
```

Lo que confirmaba que la instalación de paquetes había comenzado normalmente.

## Seguimiento de la actualización

Se comprobó el progreso mediante:

```bash
tail -f /var/log/apt/term.log
```

Las últimas líneas mostraban:

```text
Processing triggers for libc-bin ...
Processing triggers for systemd ...
Log ended ...
```

Mensajes típicos del final de una actualización satisfactoria.

## Verificaciones finales

### Comprobación de paquetes pendientes

```bash
apt -f install
```

Resultado:

```text
Upgrading: 0, Installing: 0, Removing: 0, Not Upgrading: 0
```

No quedaba nada pendiente.

### Auditoría de dpkg

```bash
dpkg --audit
```

Sin salida.

Esto significa que no hay paquetes:

- medio instalados,
- medio configurados,
- ni inconsistencias en la base de datos de `dpkg`.

### Verificación de versiones

```bash
pveversion -v
```

Mostraba:

```text
pve-manager: 9.2.20
```

confirmando que la actualización de Proxmox se había aplicado correctamente.

También se comprobó el kernel activo:

```bash
uname -r
```

Resultado:

```text
7.0.14-15-pve
```

Mientras que ya estaba instalado el nuevo:

```text
7.0.14-19-pve
```

Por tanto sólo faltaba reiniciar para arrancar con el kernel actualizado.

## Conclusión

La actualización nunca se interrumpió realmente.

Lo que ocurrió fue:

1. La consola web de Proxmox se perdió debido a un corte de red.
2. `apt-get` quedó esperando a `apt-listchanges`.
3. El paginador perdió su terminal y nunca recibió la pulsación necesaria para continuar.
4. Se identificó el bloqueo analizando el árbol de procesos.
5. Se cerró únicamente el paginador.
6. La actualización continuó desde el mismo punto.
7. Se verificó que `dpkg` terminó correctamente.
8. Se confirmó que Proxmox quedó actualizado a la versión 9.2.20.

La clave fue **no tocar los locks ni matar `apt-get` o `dpkg`**, sino localizar exactamente qué proceso estaba esperando interacción del usuario y desbloquear únicamente ese punto.

## Lección aprendida

Cuando una actualización de Proxmox parece haberse quedado colgada tras perder una sesión web, lo primero no debe ser eliminar locks ni matar procesos.

Conviene identificar el proceso que mantiene el bloqueo, inspeccionar su árbol y comprobar si está esperando interacción del usuario.

En este caso, la actualización se recuperó sin ningún riesgo porque el problema no estaba en `dpkg`, sino en un paginador huérfano de `apt-listchanges`.

### Historial de cambios

* **2026-09-29**: Documento inicial
