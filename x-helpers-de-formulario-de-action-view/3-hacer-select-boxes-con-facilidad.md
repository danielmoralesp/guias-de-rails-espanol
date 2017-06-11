# 3- Hacer Select Boxes con facilidad

Las casillas de selección en HTML requieren una cantidad significativa de etiquetas \(un elemento `OPTION` para cada opción a elegir\), por lo tanto, tiene más sentido que se generen dinámicamente.

Así es como podría lucir el etiquetado:

```ruby
<select name="city_id" id="city_id">
  <option value="1">Lisbon</option>
  <option value="2">Madrid</option>
  ...
  <option value="12">Berlin</option>
</select>
```

Aquí tiene una lista de ciudades cuyos nombres se le presentan al usuario. Internamente, la aplicación sólo quiere manejar sus `IDs` para que se usen como el atributo `value` de las opciones. Veamos cómo Rails nos puede ayudar aquí.

### 3.1 Las etiquetas de select y option

El ayudante más genérico es `select_tag`, que -como su nombre indica- simplemente genera la etiqueta `SELECT` que encapsula una cadena de opciones:

```ruby
<%= select_tag(:city_id, '<option value="1">Lisbon</option>...') %>
```

Esto es un comienzo, pero no crea dinámicamente las etiquetas `option`. Puede generar etiquetas `option` con el helper `options_for_select`:

```ruby
<%= options_for_select([['Lisbon', 1], ['Madrid', 2], ...]) %>

output:

<option value="1">Lisbon</option>
<option value="2">Madrid</option>
...
```

El primer argumento a `options_for_select` es un array anidado donde cada elemento tiene dos elementos `:text` de la opción \(nombre de la ciudad\) y valor de la opción \(`id` de la ciudad\). El valor de la opción es lo que se enviará a su controlador. A menudo, este será el `id` de un objeto de la base de datos correspondiente, pero este no tiene que ser siempre el caso.

Sabiendo esto, puede combinar `select_tag` y `options_for_select` para lograr el etiquetado completo deseado:

```ruby
<%= select_tag(:city_id, options_for_select(...)) %>
```

`options_for_select` le permite pre-seleccionar una opción pasando su valor.

```ruby
<%= options_for_select([['Lisbon', 1], ['Madrid', 2], ...], 2) %>

output:

<option value="1">Lisbon</option>
<option value="2" selected="selected">Madrid</option>
...
```

Siempre que Rails vea que el valor interno de una opción generada coincide con este valor, agregará el atributo `selected` a esa opción.

Cuando `:include_blank` o `:prompt` no están presentes, `include_blank` es forzado a `true` si el atributo de select `required` es `true`, el `size` de la pantalla es uno y `multiple` no es `true`.

Puede agregar atributos arbitrarios a las opciones utilizando hashes:

```ruby
<%= options_for_select(
  [
    ['Lisbon', 1, { 'data-size' => '2.8 million' }],
    ['Madrid', 2, { 'data-size' => '3.2 million' }]
  ], 2
) %>

output:

<option value="1" data-size="2.8 million">Lisbon</option>
<option value="2" selected="selected" data-size="3.2 million">Madrid</option>
...
```

### 3.2 Select box para tratar con modelos

En la mayoría de los casos, los controles de formulario estarán vinculados a un modelo de base de datos específico y, como cabría esperar, Rails proporciona ayuda adaptada para ese propósito. De acuerdo con otros helpers de formulario, cuando se trata de modelos se dejaría el sufijo `_tag` de `select_tag`:

```ruby
# controller:
@person = Person.new(city_id: 2)
```

```ruby
# view:
<%= select(:person, :city_id, [['Lisbon', 1], ['Madrid', 2], ...]) %>
```

Observe que en el tercer parámetro, el array de opciones, es el mismo tipo de argumento que pasa a `options_for_select`. Una ventaja aquí es que no tiene que preocuparse por la preselección de la ciudad correcta si el usuario ya tiene una - Rails lo hará por usted leyendo desde el atributo `@person.city_id`.

Al igual que con otros helpers, si utilizas el helper select en un constructor de formularios con un alcance al objeto `@person`, la sintaxis sería:

```ruby
# select on a form builder
<%= f.select(:city_id, ...) %>
```

También puede pasar un bloque para seleccionar el helper:

```ruby
<%= f.select(:city_id) do %>
  <% [['Lisbon', 1], ['Madrid', 2]].each do |c| -%>
    <%= content_tag(:option, c.first, value: c.last) %>
  <% end %>
<% end %>
```

> Si está usando select \(o helpers similares como `collection_select`, `select_tag`\) para establecer una asociación `belongs_to` debe pasar el nombre de la clave fornaea \(en el ejemplo anterior `city_id`\), no el nombre de la asociación en sí. Si especifica `city` en lugar de `city_id` Active Record generará un error a lo largo de las líneas de ActiveRecord :: `ActiveRecord::AssociationTypeMismatch: City(#17815740) expected, got String(#1138750)` al pasar el `params` hash a `Person.new` o `update`. Otra forma de ver esto es que los helpers de formulario sólo editan los atributos. También debe tener en cuenta las potenciales ramificaciones de seguridad de permitir a los usuarios editar claves foráneas directamente.





### 3.3 Option Tags de una colección de objetos arbitrarios

La generación de etiquetas de opciones con `options_for_select` requiere que se cree un array que contenga el texto y el valor de cada opción. Pero ¿qué pasaría si tuvieras un modelo de `City` \(quizás uno de Active Record\) y quisieras generar etiquetas de opciones de una colección de esos objetos? Una solución sería hacer una matriz anidada iterando sobre ellos:

```ruby
<% cities_array = City.all.map { |city| [city.name, city.id] } %>
<%= options_for_select(cities_array) %>
```

Esta es una solución perfectamente válida, pero Rails proporciona una alternativa menos verbosa: `options_from_collection_for_select`. Este ayudante espera una colección de objetos arbitrarios y dos argumentos adicionales: los nombres de los métodos para leer el valor y texto de la opción, respectivamente:

```ruby
<%= options_from_collection_for_select(City.all, :id, :name) %>
```

Como su nombre lo indica, esto sólo genera etiquetas de opción. Para generar un cuadro de selección que funcione, necesitará utilizarlo junto con `select_tag`, tal como lo haría con `options_for_select`. Cuando se trabaja con objetos de un modelo, de la misma manera que `select` combina `select_tag` y `options_for_select`, `collection_select` combina `select_tag` con `options_from_collection_for_select`.

```ruby
<%= collection_select(:person, :city_id, City.all, :id, :name) %>
```

Al igual que con otros helpers, si utilizas el helper `collection_select` en un constructor de formularios con un scope en el objeto `@person`, la sintaxis sería:

```ruby
<%= f.collection_select(:city_id, City.all, :id, :name) %>
```

Para recapitular, `options_from_collection_for_select` es un `collection_select` que `options_for_select` debe seleccionar.

> Los pares pasados a `options_for_select` deben tener el `name` primero y el `id` segundo, sin embargo, con `options_from_collection_for_select` el primer argumento es el método `value` y el segundo el método `text`.





### 3.4 Zona Horaria y Selección de País

Para aprovechar el soporte de zona horaria en Rails, debe preguntar a los usuarios en qué zona horaria se encuentran. Debería generar opciones de selección de una lista de objetos `TimeZone` predefinidos mediante `collection_select`, pero  puede usar simplemente el ayudante `time_zone_select` que ya envuelve esto:

```ruby
<%= time_zone_select(:person, :time_zone) %>
```

También hay un helper `time_zone_options_for_select` para una forma más manual \(por lo tanto más personalizable\) de hacer esto. Lea la documentación de la API para conocer los posibles argumentos para estos dos métodos.

Rails solía tener un ayudante `country_select` para elegir países, pero esto se ha extraído en el plugin `country_select`. Al usar esto, tenga en cuenta que la exclusión o inclusión de ciertos nombres de la lista puede ser algo controversial \(y fue la razón de que esta funcionalidad se extrajo de Rails\).









