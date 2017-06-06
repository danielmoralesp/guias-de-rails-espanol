# 6.9 FormOptionsHelper

Proporciona una serie de métodos para convertir diferentes tipos de contenedores en un conjunto de etiquetas de opción.



### 6.9.1 collection\_select

Devuelve las etiquetas select y option para la colección de valores de retorno existentes del método para la clase del objeto.

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



