# 6.4 CacheHelper



### 6.4.1 cache

Un método para almacenar en caché fragmentos de una vista en lugar de toda una acción o página. Esta técnica es útil para almacenar en caché piezas como menús, listas de temas de noticias, fragmentos `HTML` estáticos, etc. Este método toma un bloque que contiene el contenido que desea almacenar en caché. Vea `AbstractController::Caching::Fragments` para obtener más información.

```ruby
<% cache do %>
  <%= render "shared/footer" %>
<% end %>
```



