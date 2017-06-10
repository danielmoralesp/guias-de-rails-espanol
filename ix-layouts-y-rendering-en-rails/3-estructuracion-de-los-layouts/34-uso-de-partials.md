# 3.4 Uso de Partials

Las plantillas parciales, usualmente llamadas "partials", son otro dispositivo para romper el proceso de renderizado en bloques más manejables. Con un partial, puede mover el código para representar una pieza particular de una respuesta a su propio archivo.



### 3.4.1 Nombrar parciales

Para renderizar un parcial como parte de una vista, utilice el método `render` dentro de la vista:

```ruby
<%= render "menu" %>
```

Esto renderizará un archivo denominado `_menu.html.erb` en ese punto dentro de la vista que se está procesando. Tenga en cuenta el carácter de subrayado principal: los partials se nombran con un carácter de subrayado principal para distinguirlos de las vistas regulares, aunque se consulten sin el subrayado. Esto es válido incluso cuando está extrayendo una parte de otra carpeta:

```ruby
<%= render "shared/menu" %>
```

Ese código extraerá el partial de `app/views/shared/_menu.html.erb`.



### 3.4.2 Uso de partials para simplificar vistas

Una forma de utilizar partials es tratarlos como el equivalente de subrutinas: como una forma de mover los detalles de una vista para que pueda captar lo que está pasando más fácilmente. Por ejemplo, podría tener una vista que se vea así:

```ruby
<%= render "shared/ad_banner" %>
 
<h1>Products</h1>
 
<p>Here are a few of our fine products:</p>
...
 
<%= render "shared/footer" %>
```

En este caso, los partials `_ad_banner.html.erb` y `_footer.html.erb` pueden tener contenido compartido por muchas páginas de la aplicación. No necesita ver los detalles de estas secciones cuando se está concentrando en una página en particular.

Como se ve en las secciones anteriores de esta guía, `yield` es una herramienta muy poderosa para limpiar sus layouts. Tenga en cuenta que es Ruby puro, por lo que puede utilizarlo casi en todas partes. Por ejemplo, podemos usarlo para no repetir código en  definiciones de diseños de formulario para varios recursos similares:

users/index.html.erb

```ruby
<%= render "shared/search_filters", search: @q do |f| %>
  <p>
    Name contains: <%= f.text_field :name_contains %>
  </p>
<% end %>
```

roles/index.html.erb

```ruby
<%= render "shared/search_filters", search: @q do |f| %>
  <p>
    Title contains: <%= f.text_field :title_contains %>
  </p>
<% end %>
```

shared/\_search\_filters.html.erb

```ruby
<%= form_for(search) do |f| %>
  <h1>Search form:</h1>
  <fieldset>
    <%= yield f %>
  </fieldset>
  <p>
    <%= f.submit "Search" %>
  </p>
<% end %>
```

Para el contenido que se comparte entre todas las páginas de la aplicación, puede utilizar parciales directamente desde los layouts.



### 3.4.3 Partial Layouts

Un parcial puede utilizar su propio archivo de layout, tal como una vista puede utilizar un layout. Por ejemplo, puede llamar a un parcial como este:

```ruby
<%= render partial: "link_area", layout: "graybar" %>
```

Esto buscaría un parcial denominado `_link_area.html.erb` y lo renderizaría usando el layout `_graybar.html.erb`. Tenga en cuenta que los layouts para parciales siguen el mismo nombre de subrayado principal que los parciales regulares y se colocan en la misma carpeta con el parcial al que pertenecen \(no en la carpeta master de los payouts\).

También tenga en cuenta que se debe especificar explícitamente `:partial` al pasar opciones adicionales como `:layout.`



### 3.4.4 Pasando variables locales

También puede pasar variables locales a parciales, haciéndolas aún más potentes y flexibles. Por ejemplo, puede utilizar esta técnica para reducir la duplicación entre páginas `new` y `edit`, manteniendo al mismo tiempo un poco de contenido distinto:

new.html.erb

```ruby
<h1>New zone</h1>
<%= render partial: "form", locals: {zone: @zone} %>
```

edit.html.erb

```ruby
<h1>Editing zone</h1>
<%= render partial: "form", locals: {zone: @zone} %>
```

\_form.html.erb

```ruby
<%= form_for(zone) do |f| %>
  <p>
    <b>Zone name</b><br>
    <%= f.text_field :name %>
  </p>
  <p>
    <%= f.submit %>
  </p>
<% end %>
```

Aunque el mismo parcial se renderizará en ambas vistas, el ayudante de presentación de Action View devolverá "New zone" para la nueva acción y "Editing zone" para la acción de edición.

Para pasar una variable local a un parcial sólo en casos específicos, utilice `local_assigns`.

`index.html.erb`

```ruby
<%= render user.articles %>
```

`show.html.erb`

```ruby
<%= render article, full: true %>
```

`_article.html.erb`

```ruby
<h2><%= article.title %></h2>
 
<% if local_assigns[:full] %>
  <%= simple_format article.body %>
<% else %>
  <%= truncate article.body %>
<% end %>
```

De esta manera es posible utilizar el parcial sin necesidad de declarar todas las variables locales.

Cada parcial también tiene una variable local con el mismo nombre que el parcial \(menos el subrayado\). Puede pasar un objeto a esta variable local mediante la opción `:object`:

```ruby
<%= render partial: "customer", object: @new_customer %>
```

Dentro del parcial del cliente , la variable cliente se referirá a `@new_customer` desde la vista principal.

Si tiene una instancia de un modelo para renderizar en un parcial, puede usar una sintaxis abreviada:

```ruby
<%= render @customer %>
```

Suponiendo que la variable de instancia `@customer` contiene una instancia del modelo `Customer`, se utilizará `_customer.html.erb` para procesarla y pasará al cliente de variable local en el parcial que se referirá a la variable de instancia `@customer` en la vista principal.



### 3.4.5 Renderizado de colecciones

Los parciales son muy útiles para renderizar colecciones. Cuando pasa una colección a un parcial a través de la opción: `collection`, el parcial se insertará una vez para cada miembro en la colección:

index.html.erb

```ruby
<h1>Products</h1>
<%= render partial: "product", collection: @products %>
```

\_product.html.erb

```ruby
<p>Product Name: <%= product.name %></p>
```

Cuando un parcial es llamado con una colección pluralizada, entonces las instancias individuales del parcial tienen acceso al miembro de la colección que se está procesando a través de una variable llamada después del parcial. En este caso, el parcial es `_product`, y dentro del parcial `_product` , puede referirse al producto para obtener la instancia que se está procesando.

También hay una abreviatura para esto. Suponiendo que `@products` es una colección de instancias de producto, simplemente puede escribir esto en `index.html.erb` para producir el mismo resultado:

```ruby
<h1>Products</h1>
<%= render @products %>
```

Rails determina el nombre del parcial que se va a usar al mirar el nombre del modelo en la colección. De hecho, incluso puede crear una colección heterogénea y convertirlo de esta manera, y Rails elegirá el parcial adecuado para cada miembro de la colección:

index.html.erb

```ruby
<h1>Contacts</h1>
<%= render [customer1, employee1, customer2, employee2] %>
```

customers/\_customer.html.erb

```ruby
<p>Customer: <%= customer.name %></p>
```

employees/\_employee.html.erb

```ruby
<p>Employee: <%= employee.name %></p>
```

En este caso, Rails usará  el parcial de cliente o empleado  según corresponda para cada miembro de la colección.

En el caso de que la colección esté vacía, `render` volverá nil, por lo que debería ser bastante sencillo proporcionar contenido alternativo.

```ruby
<h1>Products</h1>
<%= render(@products) || "There are no products available." %>
```



### 3.4.6 Variables locales

Para utilizar un nombre de variable local personalizada dentro del parcial, especifique la opción `:as` en la llamada al parcial:

```ruby
<%= render partial: "product", collection: @products, as: :item %>
```

Con este cambio, puede acceder a una instancia de la colección `@products` como la variable local `item` dentro del parcial.

También puede pasar variables arbitrarias locales a cualquier parcial que esté procesando con los locales `:{} option`:

```ruby
<%= render partial: "product", collection: @products,
           as: :item, locals: {title: "Products Page"} %>
```

En este caso, el parcial tendrá acceso a un título de variable local con el valor "Products page".

> Rails también hace que una variable de contador esté disponible dentro de un parcial llamado por la colección, nombrado después del miembro de la colección seguido de `_counter`. Por ejemplo, si está procesando `@products`, en el parcial puede referirse a `product_counter` para indicarle cuántas veces se ha procesado el parcial. Esto no funciona junto con la opción `as: :value`.

También puede especificar un segundo parcial que se renderizará entre instancias del parcial principal mediante la opción: `spacer_template`:



### 3.4.7 Spacer Templates

```ruby
<%= render partial: @products, spacer_template: "product_ruler" %>
```

Rails hará que  `_product_ruler` sea parcial \(sin datos pasados a ella\) entre cada par de `_product` partials.



### 3.4.8 Collection Partial Layouts

Al renderizar colecciones también es posible utilizar la opción `:layout`:

```ruby
<%= render partial: "product", collection: @products, layout: "special_layout" %>
```

El diseño se renderizará junto con el parcial para cada elemento de la colección. Las variables actuales de objeto y objeto-contador  estarán disponibles en el layout, así como en el parcial.

