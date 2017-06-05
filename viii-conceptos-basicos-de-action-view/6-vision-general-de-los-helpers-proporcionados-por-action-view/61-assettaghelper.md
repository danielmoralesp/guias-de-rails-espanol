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



