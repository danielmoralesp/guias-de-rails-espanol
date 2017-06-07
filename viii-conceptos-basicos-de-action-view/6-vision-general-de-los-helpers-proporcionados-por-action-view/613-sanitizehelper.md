# 6.13 SanitizeHelper

El módulo SanitizeHelper proporciona un conjunto de métodos para borrar el texto de elementos HTML no deseados.



### 6.13.1 sanitize

Este helper sanitize codificará en HTML todas las etiquetas y quitará todos los atributos que no se permiten específicamente.

```ruby
sanitize @article.body
```

Si se pasan las opciones  `:attributes` o `:tags`, sólo se permiten los atributos y etiquetas mencionados y nada más.

```ruby
sanitize @article.body, tags: %w(table tr td), attributes: %w(id class style)
```

Para cambiar los valores predeterminados para usos múltiples, por ejemplo, agregar etiquetas de tabla al valor predeterminado:

```ruby
class Application < Rails::Application
  config.action_view.sanitized_allowed_tags = 'table', 'tr', 'td'
end
```



### 6.13.2 sanitize\_css\(style\)

Desinfecta un bloque de código CSS.



### 6.13.3 strip\_links\(html\)

Borra todas las etiquetas de enlace del texto y deja sólo el texto del enlace.

```ruby
strip_links('<a href="http://rubyonrails.org">Ruby on Rails</a>')
# => Ruby on Rails


strip_links('emails to <a href="mailto:me@email.com">me@email.com</a>.')
# => emails to me@email.com.


strip_links('Blog: <a href="http://myblog.com/">Visit</a>.')
# => Blog: Visit.
```



### 6.13.4 strip\_tags\(html\)

Elimina todas las etiquetas HTML del html, incluidos los comentarios. Esta funcionalidad es alimentada por la gema `rails-html-sanitizer`.

```ruby
strip_tags("Strip <i>these</i> tags!")
# => Strip these tags!


strip_tags("<b>Bold</b> no more!  <a href='more.html'>See more</a>")
# => Bold no more!  See more
```

Nota: La salida todavía puede contener caracteres '`<`', '`>`', '`&`' sin escape y confundir los navegadores.

