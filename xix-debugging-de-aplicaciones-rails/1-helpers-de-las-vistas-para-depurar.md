# 1- Helpers de las Vistas para depurar

Una tarea común es inspeccionar el contenido de una variable. Rails ofrece tres formas diferentes de hacer esto:

* `debug`
* `to_yaml`
* `inspect`

### 1.1 debug

El helper `debug` devolverá una etiqueta `<pre>` que procese el objeto utilizando el formato `YAML`. Esto generará datos legibles desde cualquier objeto. Por ejemplo, si tiene este código en una vista:

```ruby
<%= debug @article %>
<p>
  <b>Title:</b>
  <%= @article.title %>
</p>
```

Verás algo como esto:

```ruby
--- !ruby/object Article
attributes:
  updated_at: 2008-09-05 22:55:47
  body: It's a very helpful guide for debugging your Rails app.
  title: Rails debugging guide
  published: t
  id: "1"
  created_at: 2008-09-05 22:55:47
attributes_cache: {}
 
 
Title: Rails debugging guide
```

### 1.2 to\_yaml

Alternativamente, llamar `to_yaml` a cualquier objeto lo convierte en YAML. Puede pasar este objeto convertido al método `simple_format` helper para dar formato a la salida. Así es como `debug` hace su magia.

```ruby
<%= simple_format @article.to_yaml %>
<p>
  <b>Title:</b>
  <%= @article.title %>
</p>
```

El código anterior procesará algo como esto:

```ruby
--- !ruby/object Article
attributes:
updated_at: 2008-09-05 22:55:47
body: It's a very helpful guide for debugging your Rails app.
title: Rails debugging guide
published: t
id: "1"
created_at: 2008-09-05 22:55:47
attributes_cache: {}
 
Title: Rails debugging guide
```

### 1.3 inspect

Otro método útil para mostrar valores de objeto es inspeccionar, especialmente cuando se trabaja con arrays o hashes. Esto imprimirá el valor del objeto como una cadena. Por ejemplo:

```ruby
<%= [1, 2, 3, 4, 5].inspect %>
<p>
  <b>Title:</b>
  <%= @article.title %>
</p>
```

renderizara:

```ruby
[1, 2, 3, 4, 5]
 
Title: Rails debugging guide
```











