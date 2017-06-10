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





