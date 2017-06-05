# 5. Rutas de las Vistas

Al procesar una respuesta, el controlador necesita resolver dónde se encuentran las diferentes vistas. De forma predeterminada, sólo aparece en el directorio `app/views`.

Podemos agregar otras ubicaciones y darles una cierta prioridad cuando resolvamos rutas usando los métodos `prepend_view_path` y `append_view_path`.



### 5.1 Ruta de la vista Prepend

Esto puede ser útil, por ejemplo, cuando queremos poner vistas dentro de un directorio diferente para subdominios.

Podemos hacer esto usando:

```ruby
prepend_view_path "app/views/#{request.subdomain}"
```

A continuación, Action View buscará primero en este directorio cuando se resuelvan las vistas.



### 5.2 Ruta de la vista Append

Del mismo modo, podemos añadir rutas:

```ruby
append_view_path "app/views/direct"
```

Esto agregará `app/views/direct` al final de las rutas de búsqueda.

