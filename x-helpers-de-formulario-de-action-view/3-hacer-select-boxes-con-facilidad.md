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



