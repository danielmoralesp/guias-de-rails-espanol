# 3.2 Entendiendo Yield

En el contexto de un layout, `yield` identifica una sección donde se debe insertar el contenido de la vista. La forma más sencilla de utilizar esto es tener un solo `yield`, en el que se inserta todo el contenido de la vista que se está procesando actualmente:

```ruby
<html>
  <head>
  </head>
  <body>
  <%= yield %>
  </body>
</html>
```

También puede crear un layout con multiples regiones con `yield`:

```ruby
<html>
  <head>
  <%= yield :head %>
  </head>
  <body>
  <%= yield %>
  </body>
</html>
```

El cuerpo principal de la vista siempre renderizará el `yield` sin nombre. Para convertir el `render` en un `yield` con nombre, utilice el método `content_for`.

