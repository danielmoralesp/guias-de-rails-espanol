# 3.3 Uso del método content\_for

El método `content_for` le permite insertar contenido en un bloque `yield` nombrado en su layout. Por ejemplo, esta vista funcionaría con el layout que acabas de ver \(en el anterior segmento 3.2\):

```ruby
<% content_for :head do %>
  <title>A simple page</title>
<% end %>
 
<p>Hello, Rails!</p>
```

El resultado de renderizar esta página en el diseño proporcionado sería este `HTML`:

```ruby
<html>
  <head>
  <title>A simple page</title>
  </head>
  <body>
  <p>Hello, Rails!</p>
  </body>
</html>
```

El método `content_for` es muy útil cuando el diseño contiene regiones distintas, como barras laterales y pies de página, que deben tener sus propios bloques de contenido insertados. También es útil para insertar etiquetas que cargan archivos JavaScript o CSS específicos de la página en el encabezado de un diseño que de otra forma es genérico.

