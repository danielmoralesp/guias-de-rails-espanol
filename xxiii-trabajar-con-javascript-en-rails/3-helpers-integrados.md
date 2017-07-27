# 3- Helpers integrados

### 3.1 Elementos remotos

Rails proporciona un montón de métodos helpers de vista escritos en Ruby para ayudarle a generar HTML. A veces, usted quiere agregar un poco de Ajax a esos elementos, y Rails le ayuda en esos casos.

Debido al JavaScript no intrusivo, los "Ajax helpers" de Rails se dividen en dos partes: mitad JavaScript y mitad Ruby.

A menos que haya deshabilitado el Asset Pipeline,` rails-ujs` proporciona la mitad de JavaScript y los helpers de las vistas de Ruby regulares que añaden las etiquetas adecuadas a su DOM.

Puede leer a continuación sobre los diferentes eventos que se disparan tratando con elementos remotos dentro de su aplicación.

#### 3.1.1 form\_with

`form_with` es un helper que ayuda con la escritura de los formularios. De forma predeterminada, `form_with` asume que su formulario utilizará Ajax. Puede inhabilitar este comportamiento pasando la opción local `:form_with`.

```ruby
<%= form_with(model: @article) do |f| %>
  ...
<% end %>
```

Esto generará el siguiente HTML:

```html
<form action="/articles" method="post" data-remote="true">
  ...
</form>
```

Tenga en cuenta el `data-remote="true"`. Ahora, el formulario será enviado por Ajax en lugar de por el mecanismo de envio normal del navegador.

Es probable que no quiera sentarse allí con un `<form>` lleno, sin embargo. Es probable que desee hacer algo después de un envío exitosa. Para ello, vincule al evento `ajax:success`. En caso de error, utilice `ajax:error`. Echale un vistazo:

```js
$(document).ready ->
  $("#new_article").on("ajax:success", (e, data, status, xhr) ->
    $("#new_article").append xhr.responseText
  ).on "ajax:error", (e, xhr, status, error) ->
    $("#new_article").append "<p>ERROR</p>"
```

Obviamente, usted querrá ser un poco más sofisticado que eso, pero es un comienzo.

#### 3.1.2 link\_to

`link_to` es un helper que ayuda a generar enlaces. Tiene una opción `:remote` que puedes usar de esta manera:

```ruby
<%= link_to "an article", @article, remote: true %>
```

Que genera

```html
<a href="/articles/1" data-remote="true">an article</a>
```

Puede vincular los mismos eventos de Ajax como lo hicimos con `form_with`. He aquí un ejemplo. Supongamos que tenemos una lista de artículos que se pueden eliminar con un solo clic. Generaríamos un HTML como este:

```ruby
<%= link_to "Delete article", @article, remote: true, method: :delete %>
```

Y escribe un CoffeeScript como este:

```js
$ ->
  $("a[data-remote]").on "ajax:success", (e, data, status, xhr) ->
    alert "The article was deleted."
```

#### 3.1.3 button\_to

`button_to` es un helper que te ayuda a crear botones. Tienes una opción `remote` que puedes llamar así:

```ruby
<%= button_to "An article", @article, remote: true %>
```

Esto genera

```html
<form action="/articles/1" class="button_to" data-remote="true" method="post">
  <input type="submit" value="An article" />
</form>
```

Dado que es sólo un `<form>`, toda la información sobre `form_with` también se aplica.

### 3.2 Personalizar elementos remotos

Es posible personalizar el comportamiento de los elementos con un atributo `data-remote` sin escribir una línea de JavaScript. Tu puedes especificar atributos de datos adicionales para lograr esto.

#### 3.2.1 data-method

Activar hipervínculos siempre da como resultado una solicitud HTTP GET. Sin embargo, si su aplicación es RESTful, algunos enlaces son de hecho acciones que cambian datos en el servidor y deben realizarse con solicitudes que no sean GET. Este atributo permite marcar tales enlaces con un método explícito como "`post`", "`put`" o "`delete`".

La forma en que funciona es que, cuando se activa el enlace, construye un formulario oculto en el documento con el atributo "`action`" correspondiente al valor "`href`" del enlace y el método correspondiente al valor del `data-method` y el envio del formulario.

> Debido a que el envío de formularios con métodos `HTTP` distintos de `GET` y `POST` no es ampliamente aceptado en los navegadores, todos los demás métodos `HTTP` se envían realmente a través de `POST` con el método indicado en el parámetro `_method`. Rails detecta y compensa automáticamente esto.

#### 3.2.2 data-url y data-params

Ciertos elementos de su página no se refieren a ninguna URL, pero es posible que desee que activen llamadas Ajax. Especificar el atributo `data-url` junto con `data-remote` disparará una llamada Ajax a la URL dada. También puede especificar parámetros adicionales a través del atributo `data-params`.

Esto puede ser útil para activar una acción en los check-boxes, por ejemplo:

```html
<input type="checkbox" data-remote="true"
    data-url="/update" data-params="id=10" data-method="put">
```

#### 3.2.3 data-type

También es posible definir explícitamente el tipo de datos Ajax mientras se realizan solicitudes de elementos de datos remotos, a través del atributo `data-type`.

### 3.3 Confirmaciones

Puede solicitar una confirmación adicional del usuario agregando un atributo de confirmación de datos en los vínculos y los formularios. Se mostrará al usuario un diálogo de confirmación de `JavaScript()` que contiene el texto del atributo. Si el usuario decide cancelar, la acción no tiene lugar.

La adición de este atributo en los enlaces activará el diálogo al hacer clic y su adición en los formularios lo activará al enviar. Por ejemplo:

```ruby
<%= link_to "Dangerous zone", dangerous_zone_path,
  data: { confirm: 'Are you sure?' } %>
```

Esto genera:

```html
<a href="..." data-confirm="Are you sure?">Dangerous zone</a>
```

El atributo también se permite en los botones de envío de formularios. Esto le permite personalizar el mensaje de aviso dependiendo del botón que se activó. En este caso, no debe tener confirmación de datos en el formulario en sí.

La confirmación predeterminada utiliza un cuadro de diálogo de confirmación de JavaScript, pero puede personalizarlo escuchando el evento de confirmación, que se dispara justo antes de que aparezca la ventana de confirmación para el usuario. Para cancelar esta confirmación predeterminada, haga que el controlador de confirmación devuelva `false`.

### 3.4 Desactivación automática

También es posible desactivar automáticamente una entrada mientras el formulario se está enviando utilizando el atributo `data-disable-with`. Esto es para evitar doble clic accidental del usuario, lo que podría dar lugar a duplicar las solicitudes HTTP que el backend no puede detectar como tal. El valor del atributo es el texto que se convertirá en el nuevo valor del botón en su estado deshabilitado.

Esto también funciona para los enlaces con el atributo `data-method`.

Por ejemplo:

```ruby
<%= form_with(model: @article.new) do |f| %>
  <%= f.submit data: { "disable-with": "Saving..." } %>
<%= end %>
```

Esto genera un formulario con:

```html
<input data-disable-with="Saving..." type="submit">
```











