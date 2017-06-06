# 6.5 CaptureHelper



### 6.5.1 capture

El método de captura le permite extraer parte de una plantilla en una variable. A continuación, puede utilizar esta variable en cualquier parte de las plantillas o del layout.

```ruby
<% @greeting = capture do %>
  <p>Welcome! The date and time is <%= Time.now %></p>
<% end %>
```

La variable capturada puede utilizarse en cualquier otro lugar.

```ruby
<html>
  <head>
    <title>Welcome!</title>
  </head>
  <body>
    <%= @greeting %>
  </body>
</html>
```



### 6.5.2 content\_for

Llamar `content_for` almacena un bloque de marcado en un identificador para uso posterior. Puede realizar llamadas posteriores al contenido almacenado en otras plantillas o el layout, pasando el identificador como argumento `yield`.

Por ejemplo, digamos que tenemos un diseño de aplicación estándar, pero también una página especial que requiere cierto código JavaScript que el resto del sitio no necesita. Podemos usar `content_for` para incluir este JavaScript en nuestra página especial sin engordar el resto del sitio.

`app/views/layouts/application.html.erb`

```ruby
<html>
  <head>
    <title>Welcome!</title>
    <%= yield :special_script %>
  </head>
  <body>
    <p>Welcome! The date and time is <%= Time.now %></p>
  </body>
</html>
```

`app/views/articles/special.html.erb`

```ruby
<p>This is a special page.</p>
 
<% content_for :special_script do %>
  <script>alert('Hello!')</script>
<% end %>
```













