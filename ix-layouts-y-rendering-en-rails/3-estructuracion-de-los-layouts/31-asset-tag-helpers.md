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





