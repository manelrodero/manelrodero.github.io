---
layout : post
blog-width: true
title: 'Migración de WordPress a Jekyll y GitHub Pages'
date: '2020-06-06 13:14:45'
readtime: true
published: true
tags:
- Blog
author:
  display_name: Manel Rodero
cover-img: "/assets/img/blog/2020-06-06_cover.png"
thumbnail-img: ""
---

Las principales motivaciones que me han llevado a migrar este blog de [WordPress](https://es.wordpress.org/) a [Jekyll](https://jekyllrb.com/) han sido:

* En septiembre de 2019 dejé de pagar el *hosting* que tenía en [Mochahost](https://www.mochahost.com/es/) y el blog estaba un poco abandonado
* No quería volver a preocuparme por los problemas de seguridad de WordPress y, sobretodo, de sus *plugins*
* Quería que el blog cargase más rápido al usar contenido estático
* Quería tener un control de versiones de las publicaciones (sobretodo de la parte *wiki*) alojando el contenido en [GitHub Pages](https://pages.github.com/)

Los pasos que he seguido para realizar esta migración han sido los siguientes:

* [Exportar todos los datos de WordPress](https://wordpress.org/support/article/tools-export-screen/) a un fichero XML
* Hacer una *backup* completo del *hosting* de Mochahost
* Habilitar WSL en Windows 10
* Instalar una distribución de Linux ([Ubuntu 18.04](https://aka.ms/wsl-ubuntu-1804))
* Instalar Jekyll y sus requisitos previos
* Seleccionar un tema Jekyll que sea gratuito, que esté actualizado y que sea sencillo de usar
* Crear un sitio web Jekyll usando el tema anterior
* Publicar este sitio web en GitHub Pages
* Enlazar el sitio web con mi dominio [manelrodero.com](https://www.manelrodero.com/)
* Convertir los antiguos posts de WordPress a Markdown
* Verificarlos y añadirlos al sitio web

## Exportar WordPress a XML

Antes de finalizar el periodo contratado en Mochahost se utilizó la opción `Tools -> Export` de la interfaz de administración de WordPress para exportar los *posts*, páginas, categorías, etc. a un fichero XML `sitename.WordPress.2019-09-07.xml`.

## Backup del hosting

Desde el [Backup Wizard de cPanel](https://docs.cpanel.net/cpanel/files/backup-wizard/) de la interfaz de administración de Mochahost se generó un fichero `backup-9.5.2019_13-44-32_username.tar.gz` con el contenido de toda la cuenta.

También se exportó la base de datos MySQL a un fichero `username_mysql_wp_wp_20190905_819.sql.gz` para tener otra opción de recuperación del blog.

## Habilitar WSL en Windows 10

[Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/) (WSL) permite ejecutar un entorno GNU/Linux directamente en Windows sin la sobrecarga que supone utilizar una máquina virtual.

La [instalación de WSL](https://docs.microsoft.com/en-us/windows/wsl/install-win10) requiere habilitar una **Optional Feature** de Windows 10 usando el *cmdlet* `Enable-WindowsOptionalFeature` de PowerShell:

```PowerShell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```

o bien el comando nativo `dism.exe`:

```Batchfile
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /norestart
```

## Instalación de Ubuntu 18.04

Aunque Microsoft tiene disponible [Ubuntu 18.04 en la Microsoft Store](https://www.microsoft.com/en-us/p/ubuntu-1804-lts/9n9tngvndl3q), en este paso a paso se utilizará la instalación manual.

Para [instalar manualmente Ubuntu 18.04](https://docs.microsoft.com/en-us/windows/wsl/install-on-server), únicamente es necesario descargar el fichero ZIP correspondiente a la distribución, descomprimirlo y ejecutarlo:

* Descargar [https://aka.ms/wsl-ubuntu-1804](https://aka.ms/wsl-ubuntu-1804)
* Extraer el archivo `Expand-Archive .\wsl-ubuntu-1804.zip D:\SOFT\WSL\wsl-ubuntu-1804 -Force`
* Ir al directorio `D:\SOFT\WSL\wsl-ubuntu-1804`
* Ejecutar `ubuntu1804.exe` para comenzar el proceso de instalación en el mismo directorio

> **Nota**: Esta "instalación" consiste en la extracción del sistema de ficheros de Ubuntu 18.04 (`rootfs`) y la creación de un directorio temporal (`temp`) para el sistema

* Introducir un **nombre de usuario** y una **contraseña** (serán necesarios a la hora de ejecutar comandos `sudo`)

### Actualizar la distribución

Para asegurar que los repositorios y los paquetes de Ubuntu estén actualizados, se ejecuta el siguiente comando:

```Shell
sudo apt-get update -y && sudo apt-get upgrade -y
```

### Backup de WSL

A continuación se realiza una copia de seguridad de WSL para poder volver a este punto si hay algún problema con la instalación de Ruby, Jekyll, etc.:

```Batchfile
wsl.exe --export Ubuntu-18.04 backup.tar
```

## Instalación de Jekyll

La [instalación de Jekyll en WSL](https://jekyllrb.com/docs/installation/windows/#installation-via-bash-on-windows-10) es tan fácil como realizarla en Ubuntu. Lo único que hay que tener en cuenta es haber instalado previamente los siguientes requisitos:

* [Ruby 2.4.0](https://www.ruby-lang.org/) o superior (incluyendo los *development headers*)
* [RubyGems](https://rubygems.org/)
* [GCC](https://gcc.gnu.org/) y [Make](https://www.gnu.org/software/make/)

A la hora de instalar los paquetes anteriores, es recomendable conocer las [dependencias de GitHub Pages](https://pages.github.com/versions/) para instalar las mismas versiones y no tener problemas de compatibilidad.

> **Nota**: A fecha 6 de junio de 2020, GitHub Pages utiliza Jekyll 3.8.7 y Ruby 2.5.8.

### Instalar Ruby

Se puede [instalar Ruby desde BrightBox](https://www.brightbox.com/docs/ruby/ubuntu/), un repositorio que contiene versiones de Ruby optimizadas para Ubuntu, de la siguiente manera:

```Shell
sudo apt-add-repository ppa:brightbox/ruby-ng -y
sudo apt-get update -y
sudo apt-get install build-essential dh-autoreconf libssl-dev zlib1g-dev liblzma-dev libgdbm-dev -y
sudo apt-get install ruby2.5 ruby2.5-dev -y
```

A fecha 6 de junio de 2020, se instalan las siguientes versiones:

```bash
$ ruby -v
ruby 2.5.7p206 (2019-10-01 revision 67816) [x86_64-linux-gnu]
$ gem -v
2.7.6.2
```

Las consideraciones a tener en cuenta sobre esta instalación son las siguientes:

* Brightbox instala la versión 2.5.7 de Ruby y no la 2.5.8
* La versión 2.5.8 de Ruby está a punto de ser EOL
* Las versiones estables de Ruby son la 2.6.6 (usada por Netlify o Foretry) y la 2.7.1
* Aunque [Jekyll 4.0](https://jekyllrb.com/news/2019/08/20/jekyll-4-0-0-released/) se liberó el 20 de agosto de 2019, GitHub Pages [no soporta Jekyll 4.0](https://github.com/github/pages-gem/issues/651) y continúa usando la versión 3.8.x
* Se podrían [usar GitHub Actions](https://sujaykundu.com/blog/post/deploy-jekyll-using-github-pages-and-github-actions#/) para utilizar Jekyll 4.x o Bundler 2.x sin depender de GitHub

### RubyGems

**RubyGems** es un **gestor de paquetes** para Ruby que proporciona un formato estándar y autocontenido (llamado *gem*) para distribuir programas o librerías.

Los comandos más típicos para gestionar las gemas son los siguientes:

* `gem --version` (versión del gestor)
* `gem install <gema>` (instala una nueva gema)
* `gem update` (actualiza las gemas)
* `gem update --system` (actualiza las gemas del sistema)
* `gem uninstall --all` (desinstala todas las gemas)

### Bundler

[**Bundler**](https://bundler.io/) permite a los proyectos Ruby instalar las versiones correctas de las gemas que necesita especificando las dependencias en un fichero `Gemfile` en la raíz del proyecto.

Un ejemplo de fichero `Gemfile` sería el siguiente:

```Ruby
source 'https://rubygems.org'
gem 'nokogiri'
gem 'rack', '~> 2.0.1'
gem 'rspec'
```

Para instalar las gemas especificados en el fichero anterior, se ejecutaría el comando `bundle install` y para actualizarlas `bundle update`.

{: .box-note}
**Nota**: Es importante conocer la [diferencia entre `bundle install` y `bundle update`](https://stackoverflow.com/questions/16495626/difference-between-bundle-install-and-bundle-update).<br><br>Los dos comandos instalan cualquier gema especificada en el fichero `Gemfile` que no esté disponible en el sistema. Pero el segundo, actualizará las gemas a la última versión disponible si no se ha especificado una concreta en `Gemfile`.<br><br>O sea que, lo mejor es usar únicamente `bundle install` (o `bundle` que es lo mismo) y dejar la actualización para cuando sea estrictamente necesario.

### Gemas de usuario

Aunque en la instalación de Jekyll en WSL no lo indican, en las instrucciones para Ubuntu recomiendan evitar la instalación de las gemas como usuario `root`.

Ésto se consigue creando un directorio propio donde instalarlas ejecutando los siguientes comandos:

```Shell
echo '' >> ~/.bashrc
echo '# Instalar Ruby Gems en ~/gems' >> ~/.bashrc
echo 'export GEM_HOME=$HOME/gems' >> ~/.bashrc
echo 'export PATH=$HOME/gems/bin:$PATH' >> ~/.bashrc
cat ~/.bashrc
source ~/.bashrc
```

### Instalar Jekyll y Bundler

Una vez se han actualizado las gemas, se puede proceder a instalar el generador de páginas **Jekyll** y el gestor de paquetes/gemas **Bundler** ejecutando los comandos:

```Shell
gem install jekyll -v 3.8.7
gem install bundler
```

A fecha 6 de junio de 2020, se instalan las siguientes versiones

```bash
$ jekyll -v
jekyll 3.8.7
$ bundler -v
Bundler version 2.1.4
```

{: .box-note}
**Nota**: Para evitar los mensajes '*Insecure world writable dir /home/manel/gems/bin in PATH, mode 040777*' hay que ejecutar el comando `chmod go-w $HOME/gems/bin`.

### Test de Jekyll

Para comprobar que Jekyll se ha instalado correctamente, se puede crear un proyecto de la siguiente manera:

```Shell
cd
jekyll new jekyll-test
cd jekyll-test
```

Como el proyecto Jekyll utiliza un fichero `Gemfile` entonces se debe generar y servir usando `bundle exec` de la siguiente manera:

```Shell
bundle install
bundle exec jekyll serve
```

Si no utilizase un fichero `Gemfile`, el proyecto Jekyll se podría servir simplemente ejecutando `jekyll serve`.

Si toda funciona correctamente, las páginas estáticas generadas se pueden ver accediendo mediante un navegador web al servidor Jekyll en [http://127.0.0.1:4000/](http://127.0.0.1:4000/).

## Seleccionar un tema

El mundo de los temas Jekyll es similar al de las plantillas de WordPress, hay para todos los gustos y colores.

En este caso he seleccionado un par de temas que parecen sencillos de usar y se han actualizado recientemente:

* [Beautiful Jekyll](https://beautifuljekyll.com/) (versión 3.0.0 del 7 de mayo de 2020)
* [Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/) (versión 4.19.3 del 6 de junio de 2020)

### Beautiful Jekyll

[Beautiful Jekyll](https://beautifuljekyll.com/) es un tema Jekyll creado por [Dean Attali](https://twitter.com/daattali) compatible con GitHub Pages.

Se puede clonar y probar el tema [Beautiful Jekyll](https://github.com/daattali/beautiful-jekyll) de GitHub Pages mediante los siguientes comandos:

```
cd
git clone https://github.com/daattali/beautiful-jekyll.git
cd beautiful-jekyll
bundle install
bundle exec jekyll serve
```

También se puede clonar una página demo que utiliza el tema de forma remota (`remote_theme`) usando un repositorio de GitHub tal como se explica en este [procedimiento avanzado](https://beautifuljekyll.com/getstarted/#install-steps-hard):

```
cd
git clone https://github.com/daattali/beautiful-jekyll-demo.git
cd beautiful-jekyll-demo
bundle install
bundle exec jekyll serve
```

### Minimal Mistakes

[Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/) es un tema Jekyll [basado en gemas](http://jekyllrb.com/docs/themes/) creado por [Michael Rose](https://twitter.com/mmistakes) compatible con GitHub Pages cuando se utiliza como tema remoto.

Para usarlo como **tema remoto** hay que:

* Crear un `Gemfile` con el siguiente contenido:

```
source "https://rubygems.org"
gem "github-pages", group: :jekyll_plugins
```
<p></p>

* Añadir el plugin `jekyll-include-cache` al *array* de *plugins* en el fichero `_config.yml`
* Obtener y actualizar las gemas ejecutando el comando `bundle install`
* Añadir el tema remoto `remote_theme: "mmistakes/minimal-mistakes"` al fichero `_config.yml`

{: .box-note}
**Nota**: Si es necesario, se puede especificar una versión concreta del tema (por ejemplo `mmistakes/minimal-mistakes@4.19.3`).

Existe una **plantilla para GitHub Pages** que se puede clonar y adaptarla reemplazando el contenido de ejemplo:

```Shell
cd
git clone https://github.com/mmistakes/mm-github-pages-starter.git
cd mm-github-pages-starter
bundle install
bundle exec jekyll serve
```

También se puede clonar el repositorio del tema para usarlo de forma local:

```Shell
cd
git clone https://github.com/mmistakes/minimal-mistakes.git
cd minimal-mistakes
bundle install
bundle exec jekyll serve
```

## GitHub Pages

Para crear un [sitio Jekyll](https://help.github.com/en/github/working-with-github-pages/setting-up-a-github-pages-site-with-jekyll) en [GitHub Pages](https://pages.github.com/) hay que hacer lo siguiente:

* Acceder a GitHub
* Crear un nuevo repositorio llamado `username.github.io` (donde `username` es nuestro usuario de GitHub)
    * Sin descripción
    * Sin README
    * Sin fichero `.gitignore`
    * Sin licencia CC
* Conectarlo con un repositorio local:

```Shell
git remote add origin https://github.com/username/username.github.io.git
git config user.email "username@mail.com"
git config user.name "username"
git add .
git commit -m "Site inicial con Jekyll GitHub Pages"
git push -u origin master
```

* Crear un fichero `Gemfile` con el siguiente contenido para usar GitHub Pages:

```Ruby
source 'https://rubygems.org'
gem "github-pages", group: :jekyll_plugins
```

* Instalar las dependencias de la [gema GitHub Pages](https://github.com/github/pages-gem) ejecutando `bundle install`
* Servir el sitio usando `bundle exec jekyll serve`
* Acceder a [http://127.0.0.1:4000/](http://127.0.0.1:4000/) para verlo localmante
* Acceder a [http://username.github.io/](http://username.github.io/) para verlo en GitHub Pages

## Crear una *site* Jekyll para este blog

Para crear la site Jekyll de este blog **desde cero** se comienza ejecutando los siguientes comandos:

```Shell
cd
jekyll new manelrodero.github.io
cd manelrodero.github.io
```

Ésto creará la siguiente estructura de ficheros y directorios:

```
-rw-r--r-- 1 manel manel   35 Jun  6 18:11 .gitignore
-rw-r--r-- 1 manel manel  398 Jun  6 18:11 404.html
-rw-rw-rw- 1 manel manel 1119 Jun  6 18:11 Gemfile
-rw-rw-rw- 1 manel manel 1837 Jun  6 18:11 Gemfile.lock
-rw-r--r-- 1 manel manel 1652 Jun  6 18:11 _config.yml
drwxrwxrwx 1 manel manel 4096 Jun  6 18:11 _posts
-rw-r--r-- 1 manel manel  539 Jun  6 18:11 about.md
-rw-r--r-- 1 manel manel  175 Jun  6 18:11 index.md
```

El fichero `.gitignore` contiene las exclusiones por defecto para un proyecto Jekyll:

```
_site
.sass-cache
.jekyll-cache
.jekyll-metadata
vendor
```

Como se puede comprobar, se excluye el directorio `_site` ya que [GitHub Pages entiende Jekyll](https://stackoverflow.com/questions/31871433/why-put-the-site-directory-of-a-jekyll-site-in-gitignore) y no hace falta subir el *site* compilado.

La estructura de un sitio Jekyll es la siguiente:

* `_site`: contiene el sitio estático generado
* `_data`: sirve para separar datos del código (por ejemplo, para un menú de navegación)
* `_layouts`: sirve para crear estilos de página
* `_includes`: sirve para incluir elementos que se repiten (por ejemplo, para un menú de navegación)
* `assets`: contiene los CSS, imágenes, JavaScript, etc.
* `_sass`: es una extensión de CSS incluida en Jekyll
* `_posts`: contiene las publicaciones de un blog

### Usar Beautiful Jekyll (local)

Para usar el tema Beautiful Jekyll de forma local hay que copiar algunos ficheros y directorios del [tema](https://github.com/daattali/beautiful-jekyll) en el directorio del proyecto:

```
404.html
Gemfile
_config.yml
_data (dir)
_includes (dir)
_layouts (dir)
aboutme.md
assets (dir) --> css, img, js
feed.xml
index.html
staticman.yml
tags.html
```

A continuación se crea el directorio `blog` y se mueve el fichero `index.html` dentro de éste (aquí estarán los *posts* paginados del blog).

La personalización del sitio web se realiza modificando los siguientes elmentos del fichero `_config.yml`:

* title
* description
* navbar-links
* avatar
* author/name
* social-network-links
* url-pretty
* timezone
* paginate_path

Para incluir información sobre el autor, generar la página principal, etc. hay que:

* Modificar el fichero `index.md`
* Modificar el fichero `aboutme.md`
* Modificar el fichero `blog/index.html`
* Borrar el fichero `_posts/yyyy-mm-dd-welcome-to-jekyll.markdown`
* Borrar el fichero `README.md`
* Borrar el fichero `getstarted.md`
* Generar un `favicon.ico`
* Generar un `manelrodero.jpg`

Una vez hecho ésto, se instalan las dependencias y se sirve el sitio web ejecutando los siguientes comandos:

```Shell
bundle install
bundle exec jekyll serve
```

Si todo funciona correctamente, el sitio web será accesible mediante [http://127.0.0.1:4000/](http://127.0.0.1:4000/).

### Usar Beautiful Jekyll (tema remoto)

Para usar el tema Beautiful Jekyll de forma remota hay que copiar algunos ficheros y directorios del [tema](https://github.com/daattali/beautiful-jekyll-demo) en el directorio del proyecto:

```Shell
cd
mkdir manelrodero.github.io
cd manelrodero.github.io
git clone https://github.com/daattali/beautiful-jekyll-demo.git .
```

A continuación se crea el directorio `blog` y se mueve el fichero `index.html` dentro de éste (aquí estarán los *posts* paginados del blog). Hay que cambiar el *layout* de **home** a **page** y eliminar el CSS de ejemplo que tiene aplicado.

La personalización del sitio web se realiza modificando los siguientes elmentos del fichero `_config.yml`:

* title
* description
* author
* navbar-links
* avatar
* social-network-links
* url-pretty
* google_analytics
* timezone
* paginate_path

Para incluir información sobre el autor, generar la página principal, etc. hay que:

* Modificar el fichero `.gitignore`

```
Gemfile.lock
_site
.sass-cache
.jekyll-cache
.jekyll-metadata
vendor
```
<p></p>

* Crear el fichero `index.md`
* Crear el fichero `about.md`
* Modificar el fichero `blog/index.html`
* Borrar los ficheros `_posts/*.md`
* Borrar los ficheros `assets/img/*`
* Borrar el ichero `assets/css/demo.css`

Una vez hecho ésto, se instalan las dependencias y se sirve el sitio web ejecutando los siguientes comandos:

```Shell
bundle install
bundle exec jekyll serve
```

Si todo funciona correctamente, el sitio web será accesible mediante [http://127.0.0.1:4000/](http://127.0.0.1:4000/).

## Usar *Custom Domain*

{: .box-note}
**Nota**: Hay que asegurarse de añadir el dominio personalizado en GitHub Pages **antes de configurarlo en el proveedor DNS**. Si se hace alrevés, alguien podría alojar un sitio web en alguno de los subdominios.

Para [usar un dominio propio](https://help.github.com/en/github/working-with-github-pages/managing-a-custom-domain-for-your-github-pages-site) en lugar del dominio `github.io` hay que:

* Crear un fichero CNAME que contenga el nombre del dominio propio `www.manelrodero.com`
* Subir el fichero a GitHub

```Shell
git add CNAME
git commit -m "Custom DNS domain"
git push -u origin master
```

* Crear un CNAME en el proveedor de DNS para hacer que `www.manelrodero.com` apunte a `manelrodero.github.io`
* Comprobar con `dig` que se ha replicado (puede tardr un tiempo)

```Shell
dig www.manelrodero.com +nostats +nocomments +nocmd
```

Además, también se recomienda configurar un dominio **apex** (o sea la raíz de nuestro dominio) añadiendo registros **A** en el proveedor DNS que apunten a los siguientes servidores de GitHub:

```
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

## Convertir *posts* WordPress

Para convertir las publicaciones de un sitio WordPress a partir de un fichero XML se puede instalar [jekyll-import](https://import.jekyllrb.com/docs/wordpressdotcom/) mediante el siguiente comando:

```Shell
gem install jekyll-import hpricot open_uri_redirections
```

{: .box-note}
**Nota**: Si se tiene acceso *shell* a la máquina donde está alojado WordPress se puede utilizar [una versión de jekyll-import que se conecta directamente a la BD](https://import.jekyllrb.com/docs/wordpress/) para extraer las publicaciones.

La conversión de las publicaciones WordPress a formato Markdown se hace de la siguiente manera:

```Shell
cd
mkdir wordpress
cd wordpress
cp ~/sitename.WordPress.yyyy-mm-dd.xml .
ruby -r rubygems -e 'require "jekyll-import";
    JekyllImport::Importers::WordpressDotCom.run({
      "source" => "sitename.WordPress.2019-09-07.xml",
      "no_fetch_images" => false,
      "assets_folder" => "assets"
    })'
```

La ejecución del comando anterior genera los siguientes directorios:

* `_attachments`
* `_nav_menu_items`
* `_pages` (las páginas)
* `_posts` (las publicaciones del blog)
* `assets` (imágenes)

La fiabilidad de la exportación depende de si el sitio web aún está *online* ya que las imágenes se descargan desde su ubicación original.

Será necesario revisar el formato Markdown de los ficheros `*.html` de la carpeta `_posts` antes de usarlos en el sitio Jekyll. En mi caso, los principales problemas con la conversión han sido los enlaces en texto con negritas (estaban todos del revés) y las rutas de directorios Windows al perderse la barra (`\`).

{: .box-note}
**Nota**: Para renombrar todos los ficheros `*.html` a `*.md` se puede utilizar el comando `mmv \*.html \#1.md` (en Ubuntu se puede instalar usando `sudo apt-get install mmv`).

# Extras

## Otras migraciones de WP a Jekyll

* [Migrating from Wordpress to Jekyll](https://trycatch.me/migrating-from-wordpress-to-jekyll/)
* [Migrate a blog from Wordpress to Jekyll](https://blog.jmtalarn.com/migrate-blog-from-wordpress-to-jekyll/)
* [Build your own website (with Jekyll and Minimal-mistakes theme)](https://zenglix.github.io/personal_website/)

## Beautiful Jekyll Sites

* [Claudia Hauff](https://chauff.github.io/) (Associate Professor)
* [Sean Killeen](https://seankilleen.com/): Microsoft MVP y desarrollador .NET
* [Mateusz Czerniawski](https://www.mczerniawski.pl): Microsoft MVP y usuario de PowerShell
* [Jack Dougherty](https://jackdougherty.org/site-info/): Profesor del Trinity College
* [Li Zeng](https://zenglix.github.io/personal_website/): Doctorando en Yale

## Plugins y herramientas

* [jekyll-email-obfuscator](https://github.com/psmiraglia/jekyll-email-obfuscator) es una implementación de la idea [How to Hide your Email Address on Web Pages](https://www.labnol.org/internet/hide-email-address-web-pages/28364/)
* [Markdownifier](http://heckyesmarkdown.com/) es un conversor de páginas web a formato Markdown a partir de una URL
* [StackEdit](https://stackedit.io/) es un editor de Markdown que se ejecuta directamente en el navegador web
