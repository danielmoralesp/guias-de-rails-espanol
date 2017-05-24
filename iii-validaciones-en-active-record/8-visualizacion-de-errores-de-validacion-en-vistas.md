# 8. Visualización de errores de validación en vistas

Una vez que haya creado un modelo y agregado validaciones, si ese modelo se crea mediante un formulario web, probablemente desee mostrar un mensaje de error cuando una de las validaciones falla. 

Debido a que cada aplicación maneja este tipo de cosas de manera diferente, Rails no incluye ningún ayudante de vista que le ayude a generar estos mensajes directamente. Sin embargo, debido al gran número de métodos que Rails le ofrece para interactuar con las validaciones en general, es bastante fácil de construir su propio helper. Además, al generar un scaffold, Rails colocará algún ERB en el `_form.html.erb` que muestra la lista completa de errores en ese modelo. 

Asumiendo que tenemos un modelo que se ha guardado en una variable de instancia llamada `@article`, se ve así:

```ruby
<% if @article.errors.any? %>
  <div id="error_explanation">
    <h2><%= pluralize(@article.errors.count, "error") %> prohibited this article from being saved:</h2>
 
    <ul>
    <% @article.errors.full_messages.each do |msg| %>
      <li><%= msg %></li>
    <% end %>
    </ul>
  </div>
<% end %>
```

Además, si utiliza los ayudantes de formulario de Rails para generar sus formularios, cuando se produce un error de validación en un campo, generará un `<div>` adicional alrededor de la entrada.

```ruby
<div class="field_with_errors">
 <input id="article_title" name="article[title]" size="30" type="text" value="">
</div>
```

A continuación, puede generar el estilo para este `div` como quiera. El scaffold que Rails genera por defecto, por ejemplo, agrega esta regla CSS:

```css
.field_with_errors {
  padding: 2px;
  background-color: red;
  display: table;
}
```

Esto significa que cualquier campo con un error termina con un borde rojo de 2 píxeles.

