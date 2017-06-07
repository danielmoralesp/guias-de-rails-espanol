# 6.9 FormOptionsHelper

Proporciona una serie de métodos para convertir diferentes tipos de contenedores en un conjunto de etiquetas de opción.



### 6.9.1 collection\_select

Devuelve las etiquetas `select` y `option` para la colección de valores de retorno existentes del método para la clase del objeto.

Estructura de objetos de ejemplo para su uso con este método:

```ruby
class Article < ApplicationRecord
  belongs_to :author
end

class Author < ApplicationRecord
  has_many :articles
  def name_with_initial
    "#{first_name.first}. #{last_name}"
  end
end
```

Ejemplo de uso \(seleccionando el `Author` asociado para una instancia de `Article`, `@article`\):

```ruby
collection_select(:article, :author_id, Author.all, :id, :name_with_initial, { prompt: true })
```

Si `@article.author_id` es 1, esto devolverá:

```ruby
<select name="article[author_id]">
  <option value="">Please select</option>
  <option value="1" selected="selected">D. Heinemeier Hansson</option>
  <option value="2">D. Thomas</option>
  <option value="3">M. Clark</option>
</select>
```



### 6.9.2 collection\_radio\_buttons

Devuelve las etiquetas `radio_button` para la colección de valores de retorno existentes del método para la clase del objeto.

Estructura de objetos de ejemplo para su uso con este método:

```ruby
class Article < ApplicationRecord
  belongs_to :author
end
 
class Author < ApplicationRecord
  has_many :articles
  def name_with_initial
    "#{first_name.first}. #{last_name}"
  end
end
```

Ejemplo de uso \(seleccionando el `Author` asociado para una instancia de `Article`, `@article`\):

```ruby
collection_radio_buttons(:article, :author_id, Author.all, :id, :name_with_initial)
```

Si `@article.author_id` es 1, esto devolverá:

```ruby
<input id="article_author_id_1" name="article[author_id]" type="radio" value="1" checked="checked" />
<label for="article_author_id_1">D. Heinemeier Hansson</label>
<input id="article_author_id_2" name="article[author_id]" type="radio" value="2" />
<label for="article_author_id_2">D. Thomas</label>
<input id="article_author_id_3" name="article[author_id]" type="radio" value="3" />
<label for="article_author_id_3">M. Clark</label>
```



### 6.9.3 collection\_check\_boxes

Devuelve etiquetas `check_box` para la colección de valores de retorno existentes del método para la clase del objeto.

Estructura de objetos de ejemplo para su uso con este método:

```ruby
class Article < ApplicationRecord
  has_and_belongs_to_many :authors
end
 
class Author < ApplicationRecord
  has_and_belongs_to_many :articles
  def name_with_initial
    "#{first_name.first}. #{last_name}"
  end
end
```

Ejemplo de uso \(seleccionando el `Author` asociado para una instancia de `Article`, `@article`\):

```ruby
collection_check_boxes(:article, :author_ids, Author.all, :id, :name_with_initial)
```

Si `@article.author_ids` es \[1\], esto devolverá:

```ruby
<input id="article_author_ids_1" name="article[author_ids][]" type="checkbox" value="1" checked="checked" />
<label for="article_author_ids_1">D. Heinemeier Hansson</label>
<input id="article_author_ids_2" name="article[author_ids][]" type="checkbox" value="2" />
<label for="article_author_ids_2">D. Thomas</label>
<input id="article_author_ids_3" name="article[author_ids][]" type="checkbox" value="3" />
<label for="article_author_ids_3">M. Clark</label>
<input name="article[author_ids][]" type="hidden" value="" />
```



### 6.9.4 option\_groups\_from\_collection\_for\_select

Devuelve una cadena de etiquetas `option`, como `options_from_collection_for_select`, pero las agrupa mediante etiquetas `optgroup` basadas en las relaciones de objeto de los argumentos.

Estructura de objetos de ejemplo para su uso con este método:

```ruby
class Continent < ApplicationRecord
  has_many :countries
  # attribs: id, name
end
 
class Country < ApplicationRecord
  belongs_to :continent
  # attribs: id, name, continent_id
end
```

Ejemplo de uso:

```ruby
option_groups_from_collection_for_select(@continents, :countries, :name, :id, :name, 3)
```

Posible salida:

```ruby
<optgroup label="Africa">
  <option value="1">Egypt</option>
  <option value="4">Rwanda</option>
  ...
</optgroup>
<optgroup label="Asia">
  <option value="3" selected="selected">China</option>
  <option value="12">India</option>
  <option value="5">Japan</option>
  ...
</optgroup>
```

Nota: Sólo se devuelven las etiquetas `optgroup` y `option`, por lo que todavía tiene que ajustar la salida en una etiqueta de selección adecuada.



### 6.9.5 options\_for\_select

Acepta un contenedor \(hash, array, enumerable, your type\) y devuelve una cadena de etiquetas `option`.

```ruby
options_for_select([ "VISA", "MasterCard" ])
# => <option>VISA</option> <option>MasterCard</option>
```

Nota: Sólo se devuelven las etiquetas `option`, tiene que ajustar esta llamada en una etiqueta de selección HTML normal.



### 6.9.6 options\_from\_collection\_for\_select

Devuelve una cadena de etiquetas `option` que se han compilado iterando sobre la colección y asignando el resultado de una llamada al método `value` como el valor de la opción y el `text` como el texto de la opción.

```ruby
# options_from_collection_for_select(collection, value_method, text_method, selected = nil)
```

Por ejemplo, imagine un bucle que itera sobre cada persona en `@project.people` para generar un `input` tag:

```ruby
options_from_collection_for_select(@project.people, "id", "name")
# => <option value="#{person.id}">#{person.name}</option>
```

Nota: Sólo se devuelven las etiquetas option, tiene que ajustar esta llamada en una etiqueta de selección HTML normal.



### 6.9.7 select

Crea una etiqueta `select` y una serie de etiquetas `option` contenidas para el objeto y método proporcionados.

Ejemplo:

```ruby
select("article", "person_id", Person.all.collect { |p| [ p.name, p.id ] }, { include_blank: true })
```

Si `@article.person_id` es 1, esto se convertiría en:

```ruby
<select name="article[person_id]">
  <option value=""></option>
  <option value="1" selected="selected">David</option>
  <option value="2">Eileen</option>
  <option value="3">Rafael</option>
</select>
```



### 6.9.8 time\_zone\_options\_for\_select

Devuelve una cadena de etiquetas `option` para prácticamente cualquier zona horaria del mundo.



### 6.9.9 time\_zone\_select

Devuelve las etiquetas select y option para el objeto y el método dados, usando time\_zone\_options\_for\_select para generar la lista de etiquetas de opciones.

```ruby
time_zone_select( "user", "time_zone")
```



### 6.9.10 date\_field

Devuelve una etiqueta de entrada del tipo "fecha" adaptada para acceder a un atributo especificado.

```ruby
date_field("user", "dob")
```



