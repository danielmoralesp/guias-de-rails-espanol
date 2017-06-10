# 3.1 Asset Tag Helpers

Los assets helper tags \(etiquetas helper de assets\) proporcionan métodos para generar `HTML` que vinculan vistas a feeds, JavaScript, hojas de estilo, imágenes, vídeos y audios. Existen seis etiquetas helper de assets disponibles en Rails:

* `auto_discovery_link_tag`
* `javascript_include_tag`
* `stylesheet_link_tag`
* `image_tag`
* `video_tag`
* `audio_tag`

Puede utilizar estas etiquetas en los layouts o en otras vistas, aunque la etiqueta `auto_discovery_link_tag`, `javascript_include_tag` y `stylesheet_link_tag` se utilizan con mayor frecuencia en la sección `<head>` de un layout.

> Las etiquetas helper de assets no verifican la existencia de los assets en las ubicaciones especificadas; Simplemente asumen que sabes lo que estás haciendo y generan el enlace.

### 3.1.1 Vinculación a Feeds con la etiqueta auto\_discovery\_link\_tag

El ayudante `auto_discovery_link_tag` genera `HTML` que la mayoría de los navegadores y lectores de feeds pueden utilizar para detectar la presencia de feeds RSS o Atom. Se necesita el tipo de enlace \(`:rss` o `:atom`\), un hash de opciones que se pasan a `url_for` y un hash de opciones para la etiqueta:

```ruby
<%= auto_discovery_link_tag(:rss, {action: "feed"},
  {title: "RSS Feed"}) %>
```

Hay tres opciones de etiqueta disponibles para la etiqueta `auto_discovery_link_tag`:

* `:rel` especifica el valor `rel` en el enlace. El valor predeterminado es "alternate".
* `:type` especifica un tipo MIME explícito. Rails generará automáticamente un tipo MIME apropiado.
* `:title` especifica el título del enlace. El valor predeterminado es el valor del `type` en mayúscula: por ejemplo, "`ATOM`" o "`RSS`".

### 3.1.2 Vinculación a archivos JavaScript con javascript\_include\_tag

El helper `javascript_include_tag` devuelve una etiqueta de script `HTML` para cada source proporcionada.

Si está utilizando Rails con el assets pipeline activado, este helper generará un enlace a `/assets/javascripts/` en lugar de `public/javascripts` que se utilizó en versiones anteriores de Rails. Este enlace es servido por el assets pipeline.

Un archivo JavaScript dentro de una aplicación Rails o un motor de Rails se encuentra en una de tres ubicaciones: `app/assets`, `lib/assets` o `vendor/assets`. Estas ubicaciones se explican en detalle en la sección de organización de assets en la Guía de Assets Pipeline.

Puede especificar una ruta de acceso completa y relativa a la raíz del documento, o una URL, si lo prefiere. Por ejemplo, para vincular a un archivo JavaScript que está dentro de un directorio llamado `javascripts` dentro de uno de `app/assets`, `lib/assets` o `vendor/assets`, haría esto:

```ruby
<%= javascript_include_tag "main" %>
```

Rails emitirá una etiqueta de script como la siguiente:

```ruby
<script src='/assets/main.js'></script>
```

La petición a este asset es servida por la gema de `Sprockets`.

Para incluir multiples archivos como por ejemplo `app/assets/javascripts/main.js` y `app/assets/javascripts/columns.js` al mismo tiempo:

```ruby
<%= javascript_include_tag "main", "columns" %>
```

Para incluir `app/assets/javascripts/main.js` y `app/assets/javascripts/photos/columns.js`:

```ruby
<%= javascript_include_tag "main", "/photos/columns" %>
```

Para incluir `http://example.com/main.js`:

```ruby
<%= javascript_include_tag "http://example.com/main.js" %>
```

### 3.1.3 Vinculación a archivos CSS con la etiqueta stylesheet\_link\_tag

El ayudante `stylesheet_link_tag` devuelve una etiqueta `HTML` `<link>` para cada source proporcionada.

Si está utilizando Rails con el "Asset Pipeline" habilitado, este helper generará un enlace a `/assets/stylesheets/`. Este enlace es procesado por la gema de Sprockets. Un archivo de hoja de estilo se puede almacenar en una de tres ubicaciones: `app/assets`, `lib/assets` o `vendor/assets`.

```ruby
<%= stylesheet_link_tag "main" %>
```

Para incluir `app/assets/stylesheets/main.css` y `app/assets/stylesheets/columns.css`:

```ruby
<%= stylesheet_link_tag "main", "columns" %>
```

Para incluir `app/assets/stylesheets/main.css` y `app/assets/stylesheets/photos/columns.css`:

```ruby
<%= stylesheet_link_tag "main", "photos/columns" %>
```

Para incluir `http://example.com/main.css`:

```ruby
<%= stylesheet_link_tag "http://example.com/main.css" %>
```

De forma predeterminada, `stylesheet_link_tag` crea vínculos con `media="screen" rel="stylesheet"`. Puede anular cualquiera de estos valores predeterminados especificando una opción apropiada \(`:media`, `:rel`\):

```ruby
<%= stylesheet_link_tag "main_print", media: "print" %>
```

### 3.1.4 Vinculación a imágenes con image\_tag

El ayudante `image_tag` crea una etiqueta `HTML` `<img/>` en el archivo especificado. De forma predeterminada, los archivos se cargan desde `public/images`.

> Tenga en cuenta que debe especificar la extensión de la imagen.

```ruby
<%= image_tag "header.png" %>
```

Puede proporcionar una ruta a la imagen si lo desea:

```ruby
<%= image_tag "icons/delete.gif" %>
```

Puede proporcionar un hash de opciones `HTML` adicionales:

```ruby
<%= image_tag "icons/delete.gif", {height: 45} %>
```

Puede suministrar texto alternativo para la imagen que se utilizará si el usuario tiene imágenes desactivadas en su navegador. Si no especifica un texto `alt` explícitamente, el valor predeterminado es el nombre del archivo, en mayúsculas y sin extensión. Por ejemplo, estas dos etiquetas de imagen devolverían el mismo código:

```ruby
<%= image_tag "home.gif" %>
<%= image_tag "home.gif", alt: "Home" %>
```

También puede especificar una etiqueta de tamaño especial, en el formato "`{width}x{height}`":

```ruby
<%= image_tag "home.gif", size: "50x20" %>
```

Además de las etiquetas especiales anteriores, puede proporcionar un hash de las opciones `HTML` estándar, como `:class`, `:id` o `:name`:

```ruby
<%= image_tag "home.gif", alt: "Go Home",
                          id: "HomeImage",
                          class: "nav_bar" %>
```

### 3.1.5 Vinculación a vídeos con la etiqueta video

El helper `video_tag` crea una etiqueta `HTML 5` `<video>` en el archivo especificado. De forma predeterminada, los archivos se cargan desde `public/videos`.

```ruby
<%= video_tag "movie.ogg" %>
```

produce

```ruby
<video src="/videos/movie.ogg" />
```

Al igual que un `image_tag`, puede proporcionar una ruta, absoluta o relativa al directorio `public/videos`. Además, puede especificar el tamaño: "`#{width}x#{height}`" como una `image_tag`. Las etiquetas de vídeo también pueden tener cualquiera de las opciones `HTML` especificadas al final \(`id`, `class` y `alt`\).

La etiqueta de vídeo también admite todas las opciones de `<video>` HTML a través del hash de opciones HTML, incluyendo:

* `poster`: "`image_name.png`", proporciona una imagen para poner en lugar del video antes de que comience a reproducirse.
* `autoplay:true`, comienza a reproducir el video en carga de página.
* `loop :true`, hace un loop el video una vez que llega al final.
* `controls :true`, proporciona controles suministrados por el navegador para que el usuario interactúe con el video.
* `autobuffer :true`, el video cargará previamente el archivo para el usuario en carga de página.

También puede especificar varios videos para reproducir pasando un array de videos a la `video_tag`:

```ruby
<%= video_tag ["trailer.ogg", "movie.ogg"] %>
```

El cual produce

```ruby
<video>
  <source src="/videos/trailer.ogg">
  <source src="/videos/movie.ogg">
</video>
```



### 3.1.6 Vinculación a archivos de audio con audio\_tag

El asistente `audio_tag` crea una etiqueta `HTML 5 <audio>` en el archivo especificado. De forma predeterminada, los archivos se cargan desde `public/audios`.

```ruby
<%= audio_tag "music.mp3" %>
```

Puede proporcionar una ruta de acceso al archivo de audio si lo desea:

```ruby
<%= audio_tag "music/first_song.mp3" %>
```

También puede proporcionar un hash de opciones adicionales, tales como `:id`, `:class` etc.

Al igual que el `video_tag`, el `audio_tag` tiene opciones especiales:

* `autoplay :true`, comienza a reproducir el audio en la carga de la página
* `controls :true`, proporciona controles suministrados por el navegador para que el usuario interactúe con el audio.
* `autobuffer :true`, el audio cargará previamente el archivo para el usuario en carga de página.



