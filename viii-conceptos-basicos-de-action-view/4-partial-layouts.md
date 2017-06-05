# 4 Partial Layouts

Los parciales pueden tener sus propios layouts aplicados a ellos. Estos layouts son diferentes de los aplicados a una acción del controlador, pero funcionan de una manera similar.

Digamos que estamos mostrando un artículo en una página que debe ser envuelto en un `div` para fines de visualización. En primer lugar, crearemos un nuevo artículo:

```ruby
Article.create(body: 'Partial Layouts are cool!')
```

En la plantilla de show, renderizaremos el partial `_article` envuelto en el layout `box`:

`articles/show.html.erb`

```ruby
<%= render partial: 'article', layout: 'box', locals: { article: @article } %>
```

El layout `box` simplemente envuelve el partial  `_article` en un `div`:

`articles/_box.html.erb`

```ruby
<div class='box'>
  <%= yield %>
</div>
```

Tenga en cuenta que el layout de partial tiene acceso a la variable local del artículo que se pasó a la llamada `render`. Sin embargo, a diferencia de los layouts de la aplicación general, los layouts parciales todavía tienen el prefijo de subrayado.

También puede procesar un bloque de código dentro de un layout parcial en lugar de llamar a `yield`. Por ejemplo, si no tuviéramos el parcial `_article` , podríamos hacer esto en su lugar:

```ruby
<% render(layout: 'box', locals: { article: @article }) do %>
  <div>
    <p><%= article.body %></p>
  </div>
<% end %>
```

Suponiendo que usamos el mismo parcial `_box` de arriba, esto produciría la misma salida que el ejemplo anterior.

