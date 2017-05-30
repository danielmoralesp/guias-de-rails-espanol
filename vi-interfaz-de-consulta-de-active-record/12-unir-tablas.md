# 12. Unir tablas

Active Record proporciona dos métodos finder para especificar cláusulas `JOIN` en el `SQL` resultante: `joins` y `left_outer_joins`. Mientras que las combinaciones se deben utilizar para `INNER JOIN` o consultas personalizadas, `left_outer_joins` se utiliza para consultas utilizando `LEFT OUTER JOIN`.



### 12.1 joins

Hay varias maneras de utilizar el método joins.

#### 12.1.1 Uso de un fragmento SQL de cadena

Sólo puede suministrar el `SQL` sin especificar la cláusula `JOIN` para unir:

```ruby
Author.joins("INNER JOIN posts ON posts.author_id = authors.id AND posts.published = 't'")
```

Esto resultará en el siguiente `SQL`:

```ruby
SELECT authors.* FROM authors INNER JOIN posts ON posts.author_id = authors.id AND posts.published = 't'
```

#### 12.1.2 Uso de Array/Hash de Asociaciones Nombradas

Active Record le permite utilizar los nombres de las asociaciones definidas en el modelo como un acceso directo para especificar las cláusulas `JOIN` para esas asociaciones cuando se utiliza el método `join`.

Por ejemplo, considere los siguientes Modelos de Category, Article, Comment, Guest y Tag

```ruby
class Category < ApplicationRecord
  has_many :articles
end
 
class Article < ApplicationRecord
  belongs_to :category
  has_many :comments
  has_many :tags
end
 
class Comment < ApplicationRecord
  belongs_to :article
  has_one :guest
end
 
class Guest < ApplicationRecord
  belongs_to :comment
end
 
class Tag < ApplicationRecord
  belongs_to :article
end
```

Ahora todo lo siguiente producirá las consultas `join` esperadas utilizando `INNER JOIN`:

##### 12.1.2.1 Joining a una asociación única

```ruby
Category.joins(:articles)
```

Esto produce:

```ruby
SELECT categories.* FROM categories
  INNER JOIN articles ON articles.category_id = categories.id
```

O, en español: "devolver un objeto `Category` para todas las categorías con artículos". Tenga en cuenta que verá categorías duplicadas si más de un artículo tiene la misma categoría. Si desea categorías únicas, puede utilizar `Category.joins(:articles).distinct`.

#### 12.1.3 Joining a múltiples asociaciones

```ruby
Article.joins(:category, :comments)
```

Esto produce:

```ruby
SELECT articles.* FROM articles
  INNER JOIN categories ON articles.category_id = categories.id
  INNER JOIN comments ON comments.article_id = articles.id
```

O, en español: "devolver todos los artículos que tienen una categoría y al menos un comentario". Tenga en cuenta que los artículos con varios comentarios se mostrarán varias veces.

##### 12.1.3.1 Unir Asociaciones Anidadas \(Nivel Único\)

```ruby
Article.joins(comments: :guest)
```

Esto produce:

```ruby
SELECT articles.* FROM articles
  INNER JOIN comments ON comments.article_id = articles.id
  INNER JOIN guests ON guests.comment_id = comments.id
```

O, en español: "devolver todos los artículos que tienen un comentario hecho por un invitado."

##### 12.1.3.2 Unir Asociaciones Anidadas \(Nivel Múltiple\)

```ruby
Category.joins(articles: [{ comments: :guest }, :tags])
```

Esto produce:

```ruby
SELECT categories.* FROM categories
  INNER JOIN articles ON articles.category_id = categories.id
  INNER JOIN comments ON comments.article_id = articles.id
  INNER JOIN guests ON guests.comment_id = comments.id
  INNER JOIN tags ON tags.article_id = articles.id
```

O, en español: "devolver todas las categorías que tienen artículos, donde esos artículos tienen un comentario hecho por un invitado, y donde esos artículos también tienen una etiqueta".

#### 12.1.4 Especificación de condiciones en las joined tables

Puede especificar condiciones en las tablas unidas usando las condiciones regulares `Array` y `String`. Las condiciones de `hash` proporcionan una sintaxis especial para especificar las condiciones de las tablas unidas:

```ruby
time_range = (Time.now.midnight - 1.day)..Time.now.midnight
Client.joins(:orders).where('orders.created_at' => time_range)
```

Una sintaxis alternativa y más limpia es anidar las condiciones con hash:

```ruby
time_range = (Time.now.midnight - 1.day)..Time.now.midnight
Client.joins(:orders).where(orders: { created_at: time_range })
```

Esto encontrará todos los clientes que tienen órdenes que se crearon ayer, de nuevo utilizando una expresión `BETWEEN SQL`.



### 12.2 left\_outer\_joins

Si desea seleccionar un conjunto de registros, independientemente de si tienen o no registros asociados, puede utilizar el método `left_outer_joins`.

```ruby
Author.left_outer_joins(:posts).distinct.select('authors.*, COUNT(posts.*) AS posts_count').group('authors.id')
```

Que produce:

```ruby
SELECT DISTINCT authors.*, COUNT(posts.*) AS posts_count FROM "authors"
LEFT OUTER JOIN posts ON posts.author_id = authors.id GROUP BY authors.id
```

Lo que significa: "devolver todos los autores con su numero de posts, si tienen o no tienen posts en absoluto"

