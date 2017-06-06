# 6.1 AssetTagHelper

Este módulo proporciona métodos para generar `HTML` que vincula vistas a elementos tales como imágenes, archivos JavaScript, hojas de estilo y feeds.

De forma predeterminada, Rails enlaza con estos assets en el host actual de la carpeta pública, pero puede dirigir a Rails para enlazar con assets desde un servidor de assets dedicados configurando `config.action_controller.asset_host` en la configuración de la aplicación, normalmente en `config/environments/production.rb`. Por ejemplo, digamos que el host de assets es `assets.example.com`:

```ruby
config.action_controller.asset_host = "assets.example.com"
image_tag("rails.png") # => <img src="http://assets.example.com/images/rails.png" alt="Rails" />
```



### 6.1.1 auto\_discovery\_link\_tag

Devuelve una etiqueta de link que los navegadores y los lectores de feeds pueden usar para detectar automáticamente un feed `RSS` o `Atom`.

```ruby
auto_discovery_link_tag(:rss, "http://www.example.com/feed.rss", { title: "RSS Feed" }) # =>
  <link rel="alternate" type="application/rss+xml" title="RSS Feed" href="http://www.example.com/feed.rss" />
```



### 6.1.2 image\_path

Calcula la ruta de acceso a un recurso de imagen en el directorio `app/assets/images`. Se pasarán a través de rutas completas desde la raíz del documento. Utilizado internamente por `image_tag` para construir la ruta de la imagen.

```ruby
image_path("edit.png") # => /assets/edit.png
```

Un fingerprint \(huella digital\) se agregará al nombre de archivo si `config.assets.digest` se establece en true.

```ruby
image_path("edit.png") # => /assets/edit-2d1a2db63fc738690021fedb5a65b68e.png
```



### 6.1.3 image\_url

Calcula la URL de un recurso de imagen en el directorio `app/assets/images.` Esto llamará a `image_path` internamente y se fusionará con su host actual o con su host de assets.

```ruby
image_url("edit.png") # => http://www.example.com/assets/edit.png
```



### 6.1.4 image\_tag

Devuelve una etiqueta de imagen HTML desde el origen. El origen puede ser una ruta de acceso completa o un archivo que existe en el directorio `app/assets/images`.

```ruby
image_tag("icon.png") # => <img src="/assets/icon.png" alt="Icon" />
```



### 6.1.5 javascript\_include\_tag

Devuelve una etiqueta de secuencia de comandos HTML para cada uno de los orígenes proporcionados. Puede pasar el nombre de archivo \(la extensión `.js` es opcional\) de los archivos JavaScript que existen en el directorio `app/assets/javascripts` para su inclusión en la página actual o puede pasar la ruta completa relativa a la raíz del documento.

```ruby
javascript_include_tag "common" # => <script src="/assets/common.js"></script>
```

Si la aplicación no utiliza la assets pipeline, para incluir la biblioteca jQuery de JavaScript en su aplicación, pase :`default` como fuente. Cuando se utiliza `:default`, si existe un archivo `application.js` en el directorio `app/assets/javascripts`, también se incluirá.

```ruby
javascript_include_tag :defaults
```

También puede incluir todos los archivos JavaScript en el directorio `app/assets/javascripts` utilizando `:all` como fuente.

```ruby
javascript_include_tag :all
```

También puede almacenar en caché varios archivos JavaScript en un solo archivo, lo que requiere menos conexiones `HTTP` para descargar y puede ser mejor comprimirlo por gzip \(lo que lleva a transferencias más rápidas\). El almacenamiento en caché sólo ocurrirá si `ActionController::Base.perform_caching` está establecido en `true` \(que es el caso por defecto para el entorno de producción de Rails, pero no para el entorno de desarrollo\).

```ruby
javascript_include_tag :all, cache: true # =><script src="/javascripts/all.js"></script>
```



### 6.1.6 javascript\_path

Computa la ruta de acceso a un recurso de JavaScript en el directorio `app/assets/javascripts`. Si el nombre de archivo de origen no tiene extensión, se agregará `.js`. Se pasará a través de rutas completas desde la raíz del documento. Use internamente el `javascript_include_tag` para construir la ruta del script.

```ruby
javascript_path "common" # => /assets/common.js
```



### 6.1.7 javascript\_url

Computa la URL de un recurso de JavaScript en el directorio `app/assets/javascripts`. Esto llamará `javascript_path` internamente y se fusionará con su host actual o con su host de assets.

```ruby
javascript_url "common" # => http://www.example.com/assets/common.js
```



### 6.1.8 stylesheet\_link\_tag

Devuelve una etiqueta de enlace de hoja de estilo para los orígenes especificados como argumentos. Si no especifica una extensión, `.css` se añadirá automáticamente.

```ruby
stylesheet_link_tag "application" # => <link href="/assets/application.css" media="screen" rel="stylesheet" />
```

También puede incluir todos los estilos en el directorio de hoja de estilo utilizando `:all` como fuente:

```ruby
stylesheet_link_tag :all
```

También puede cachear varias hojas de estilo en un archivo, lo que requiere menos conexiones `HTTP` y puede ser mejor si es compreso por gzip \(lo que lleva a transferencias más rápidas\). El almacenamiento en caché sólo ocurrirá si `ActionController::Base.perform_caching` está establecido en `true` \(que es el caso por defecto para el entorno de producción de Rails, pero no para el entorno de desarrollo\).

```ruby
stylesheet_link_tag :all, cache: true # => <link href="/assets/all.css" media="screen" rel="stylesheet" />
```



### 6.1.9 stylesheet\_path

Computa la ruta de acceso a un elemento de hoja de estilo en el directorio `app/assets/stylesheets`. Si el nombre de archivo de origen no tiene extensión, se agregará `.css`. Se pasarán a través de rutas completas desde la raíz del documento. Se utiliza internamente por `stylesheet_link_tag` para crear la ruta de la hoja de estilos.

```ruby
stylesheet_path "application" # => /assets/application.css
```



### 6.1.10 stylesheet\_url

Computa la URL de un elemento de hoja de estilo en el directorio `app/assets/stylesheets`. Esto llamará a `stylesheet_path` internamente y se fusionará con su host actual o con su host activo.

```ruby
stylesheet_url "application" # => http://www.example.com/assets/application.css
```



