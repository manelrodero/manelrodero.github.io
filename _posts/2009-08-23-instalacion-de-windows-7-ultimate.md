---
layout: post
blog-width: true
title: Instalación de Windows 7 Ultimate
date: 2009-08-23 14:14:21
published: true
tags:
- Software
- Windows
author:
  display_name: Manel Rodero
---

Desde que probé las versiones Beta y RC de Windows 7 antes del verano (arrancando desde un fichero VHD usando el hardware real de mi máquina) estaba decidido a cambiar mi instalación actual de Windows Vista en cuanto apareciera la versión final RTM del nuevo sistema operativo de Microsoft. La razón fundamental es que Windows 7 funciona mucho más fino que Windows Vista (es más rápido y no consume tantos recursos).

A principios de Agosto apareció la versión RTM para los suscriptores de TechNet y ahora, después de las vacaciones, he comenzado el proceso de actualización a Windows 7 Ultimate (con la versión 64 bits y el idioma inglés por aquello de no toparte con alguna traducción pésima y facilitar la búsqueda de información).

La primera idea ha sido actualizar directamente mi instalación de Windows Vista a Windows 7 para conservar los programas instalados (pocos, la verdad sea dicha, porque me he ido pasando a las versiones "portables" de muchos programas). La actualización se ha realizado sin problemas aunque he tenido que eliminar algunos programas que Windows 7 recomendaba desinstalar antes de continuar:

* Canon EOS Utility
* iTunes
* MagicISO Virtual CD/DVD Manager
* ATI CATALYST Install Manager
* ATI Catalyst Control Center

De todas maneras, la instalación no acababa de ir todo lo fina que debería (supongo que debía haber algún problema con los drivers de la instalación de Windows Vista) por lo que, al final, he decidido hacer una instalación desde cero de Windows 7.

Antes de empezar, como medida de seguridad, he creado una máquina virtual VMWare a partir de mi instalación de Windows Vista usando VMWare Converter. Después la he probado en VMWare Workstation para comprobar que arrancaba. Esto me permitirá recuperar información sobre los programas instalados, sus configuraciones y las claves de registro de aquellos que tengo comprados (DUMeter, Macrium Reflect, etcétera).

El proceso de instalación ha sido el siguiente:

1. Grabar el fichero ISO descargado de TechNet (`en_windows_7_ultimate_x64_dvd_x15-65922.iso`) en un DVD y arrancar con él.
2. Cambiar el idioma de entrada (teclado) a **Spanish** y dejar el idioma del SO y de la configuración regional en **English**.
3. Formatear la partición C: (VISTA) para que no dejar restos de la instalación anterior e instalar en ella.
4. Dejar que finalice el proceso de copia de ficheros, instalación y configuración.

Al finalizar la instalación la pantalla se ha quedado en negro y con el cursor que podía mover perfectamente. El "problema" era debido a que la tarjeta gráfica estaba enviando la señal a través del puerto DVI donde tengo conectada mi TV LCD Philips de 42″. En mi pantalla Dell de 22″ únicamente estaba viendo el escritorio secundario. La solución ha consistido en cambiar el **Main Display** en las propiedades de resolución del escritorio. ¡Uff, qué susto!

A continuación he indicado el nombre de la cuenta de usuario inicial y el nombre del equipo. Después he configurado la fecha y hora, las actualizaciones automáticas y he introducido la **Product Key** para activar el producto cuando estemos conectados a Internet.

Finalmente, he activado la cuenta **Administrator** (la instalación de programas y la configuración de Windows 7 la realizaré desde esta cuenta) y le he rebajado los permisos a la cuenta de usuario inicial (así será más segura la navegación por Internet y se minimizan los daños ante un virus, troyano, etcétera):

1. Ejecutar `cmd.exe` como administrador.
2. `net user administrator _password_`
3. `net user administrator /active:yes`
4. `net localgroup "administrators" _usuario_ /delete`
5. `net localgroup "power users" _usuario_ /add`

Y eso es todo… a partir de ahora toca configurar el SO, instalar los programas que usaré, etcétera.
