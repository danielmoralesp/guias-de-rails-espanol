# 4.3 Referencia de la asociación has\_many

La asociación `has_many` crea una relación uno-a-muchos con otro modelo. En términos de base de datos, esta asociación dice que la otra clase tendrá una clave externa que se refiere a instancias de esta clase.



### 4.3.1 Métodos Añadidos por has\_many

Cuando declara una asociación `has_many`, la clase declarante obtiene automáticamente 16 métodos relacionados con la asociación:

```ruby
collection
collection<<(object, ...)
collection.delete(object, ...)
collection.destroy(object, ...)
collection=(objects)
collection_singular_ids
collection_singular_ids=(ids)
collection.clear
collection.empty?
collection.size
collection.find(...)
collection.where(...)
collection.exists?(...)
collection.build(attributes = {}, ...)
collection.create(attributes = {})
collection.create!(attributes = {})
```

En todos estos métodos, la palabra `collection` se reemplaza con el símbolo pasado como el primer argumento a `has_many`, y `collection_singular` se reemplaza con la versión singularizada de ese símbolo. Por ejemplo, dada la declaración:

```ruby
class Author < ApplicationRecord
  has_many :books
end
```

Cada instancia del modelo `Author` tendrá estos métodos:

```ruby
Author.books
Author.books<<(object, ...)
Author.books.delete(object, ...)
Author.books.destroy(object, ...)
Author.books=(objects)
Author.book_ids
Author.book_ids=(ids)
Author.books.clear
Author.books.empty?
Author.books.size
Author.books.find(...)
Author.books.where(...)
Author.books.exists?(...)
Author.books.build(attributes = {}, ...)
Author.books.create(attributes = {})
Author.books.create!(attributes = {})
```

#### 4.3.1.1 collection 

El método `collection` devuelve un array de todos los objetos asociados. Si no hay objetos asociados, devuelve un array vacío.

```ruby
@books = @author.books
```

#### 4.3.1.2 collection&lt;&lt;\(object, ...\) 

El método `collection<<` agrega uno o más objetos a la colección fijando sus llaves foráneas a la llave primaria del modelo llamante.

```ruby
@author.books << @book1
```

#### 4.3.1.3 collection.delete\(object, ...\) 

El método `collection.delete` elimina uno o más objetos de la colección estableciendo sus claves foráneas en `NULL`.

```ruby
@author.books.delete(@book1)
```

Además, los objetos se destruirán si están asociados con `dependent: :destroy` y se eliminan si están asociados con `dependent: :delete_all`.

#### 4.3.1.4 collection.destroy \(object, ...\) 

El método `collection.destroy` elimina uno o más objetos de la colección ejecutando `destroy` en cada objeto.

```ruby
@author.books.destroy(@book1)
```

Los objetos siempre se eliminarán de la base de datos, ignorando la opción `:dependent`.

#### 4.3.1.5 collection=\(objects\) 

El método `collection=` hace que la colección contenga sólo los objetos suministrados, agregando y eliminando según corresponda. Los cambios son persistentes en la base de datos.

#### 4.3.1.6 collection\_singular\_ids 

El método `collection_singular_ids` devuelve un array de `ids` de los objetos de la colección.

```ruby
@book_ids = @author.book_ids
```

#### 4.3.1.7 collection\_singular\_ids=\(ids\)

El método `collection_singular_ids=` hace que la colección contenga sólo los objetos identificados por los valores de clave foránea suministrados, añadiendo y eliminando según corresponda. Los cambios son persistentes en la base de datos.

#### 4.3.1.8 collection.clear 

El método `collection.clear` elimina todos los objetos de la colección de acuerdo con la estrategia especificada por la opción dependiente. Si no se da ninguna opción, sigue la estrategia predeterminada. La estrategia predeterminada para `has_many: through`  es `delete_all`, y para `has_many`  es establecer las claves externas en NULL.

```ruby
@author.books.clear
```

Los objetos se eliminarán si están asociados con `dependent: :destroy`, al igual que `dependent: :delete_all`.

#### 4.3.1.9 collection.empty?

El Método `collection.empty?` devuelve `true` si la colección no contiene ningún objeto asociado.

```ruby
<% if @author.books.empty? %>
  No Books Found
<% end %>
```

#### 4.3.1.10 collection.size 

El método `collection.size` devuelve el número de objetos de la colección.

```ruby
@book_count = @author.books.size
```

#### 4.3.1.11 collection.find\(...\) 

El método `collection.find` encuentra objetos dentro de la colección. Utiliza la misma sintaxis y opciones como `ActiveRecord::Base.find`.

```ruby
@available_books = @author.books.find(1)
```

#### 4.3.1.12 collection.where\(...\) 

El método `collection.where` encuentra objetos dentro de la colección basados en las condiciones suministradas, pero los objetos se cargan perezosamente, lo que significa que la base de datos se consulta sólo cuando se accede a los objetos.

```ruby
@available_books = @author.books.where(available: true) # No query yet
@available_book = @available_books.first # Now the database will be queried
```

#### 4.3.1.13 collection.exists?\(...\) 

El método `collection.exists?` comprueba si existe un objeto que cumple las condiciones suministradas en la colección. Utiliza la misma sintaxis y opciones como `ActiveRecord::Base.exists?`.

#### 4.3.1.14 collection.build \(attributes = {}, ...\) 

El método `collection.build` devuelve una sola o una matriz de nuevos objetos del tipo asociado. Los objetos serán instanciados a partir de los atributos pasados y el enlace a través de su clave externa se creará, pero los objetos asociados no se guardarán todavía.

```ruby
@book = @author.books.build(published_at: Time.now,
                                book_number: "A12345")
 
@books = @author.books.build([
  { published_at: Time.now, book_number: "A12346" },
  { published_at: Time.now, book_number: "A12347" }
```

#### 4.3.1.15 collection.create\(attributes = {}\) 

El método `collection.create` devuelve una sola o una matriz de nuevos objetos del tipo asociado. Los objetos serán instanciados a partir de los atributos pasados, se creará el enlace a través de su clave externa y, una vez que se pasen todas las validaciones especificadas en el modelo asociado, se guardará el objeto asociado.

```ruby
@book = @author.books.create(published_at: Time.now,
                                 book_number: "A12345")
 
@books = @author.books.create([
  { published_at: Time.now, book_number: "A12346" },
  { published_at: Time.now, book_number: "A12347" }
])
```

#### 4.3.1.16 collection.create!\(attributes = {}\) 

Hace lo mismo que el anterior `collection.create`, pero dispara `ActiveRecord::RecordInvalid` si el registro no es válido.



### 4.3.2 Opciones para has\_many 

Mientras que Rails utiliza valores predeterminados inteligentes que funcionarán bien en la mayoría de las situaciones, puede haber ocasiones en las que desee personalizar el comportamiento de la referencia de asociación `has_many`. Tales personalizaciones pueden realizarse fácilmente pasando opciones al crear la asociación. Por ejemplo, esta asociación utiliza dos opciones:

```ruby
class Author < ApplicationRecord
  has_many :books, dependent: :delete_all, validate: false
end
```

La asociación `has_many` soporta estas opciones:

```ruby
:as
:autosave
:class_name
:counter_cache
:dependent
:foreign_key
:inverse_of
:primary_key
:source
:source_type
:through
:validate
```

#### 4.3.2.1: as 

Establecer la opción: `as` indica que se trata de una asociación polimórfica, como se ha explicado anteriormente en esta guía. 

#### 4.3.2.2: autosave 

Si configura la opción: `autosave` en `true`, Rails guardará todos los miembros cargados y destruirá los miembros marcados para su destrucción cada vez que guarde el objeto principal. 

#### 4.3.2.3: class\_name 

Si el nombre del otro modelo no puede derivarse del nombre de la asociación, puede utilizar la opción: `class_name` para proporcionar el nombre del modelo. Por ejemplo, si un autor tiene muchos libros, pero el nombre real del modelo que contiene los libros es `Transaction`, configurarías las cosas de esta manera:

```ruby
class Author < ApplicationRecord
  has_many :books, class_name: "Transaction"
end
```

#### 4.3.2.4: counter\_cache 

Esta opción se puede utilizar para configurar un nombre personalizado denominado: `counter_cache`. Sólo necesita esta opción cuando personaliza el nombre de su: `counter_cache` en la asociación `belongs_to`. 

#### 4.3.2.5 :dependent 

Controla qué sucede con los objetos asociados cuando se destruye a su propietario:

* `:destroy` hace que todos los objetos asociados también sean destruidos 
* `:delete_all` hace que todos los objetos asociados sean eliminados directamente de la base de datos \(por lo que las devoluciones de llamada no se ejecutarán\) 
* `:nullify` hace que las claves externas se establezcan en `NULL`. Las devoluciones de llamada no se ejecutan. 
* `:restrict_with_exception` provoca que se genere una excepción si hay registros asociados 
* `:restrict_with_error` causa que se agregue un error al propietario si hay objetos asociados

#### 4.3.2.6: foreign\_key 

Por convención, Rails asume que la columna utilizada para mantener la clave externa en el otro modelo es el nombre de este modelo con el sufijo `_id` añadido. La opción: `foreign_key` le permite establecer el nombre de la clave externa directamente:

```ruby
class Author < ApplicationRecord
  has_many :books, foreign_key: "cust_id"
end
```

En cualquier caso, Rails no creará columnas de clave externa para usted. Debe definirlos explícitamente como parte de sus migraciones.

#### 4.3.2.7 :inverse\_of 

La opción: `inverse_of` especifica el nombre de la asociación `belongs_to` que es la inversa de esta asociación. No funciona en combinación con las opciones: `through` o: `as`.

```ruby
class Author < ApplicationRecord
  has_many :books, inverse_of: :author
end
 
class Book < ApplicationRecord
  belongs_to :author, inverse_of: :books
end
```

#### 4.3.2.8: primary\_key

Por convención, Rails asume que la columna utilizada para mantener la clave primaria de la asociación es `id`. Puede sobre-escribirlo y especificar explícitamente la clave principal con la opción: `primary_key`. 

Digamos que la tabla users tiene `id` como `primary_key` pero también tiene una columna `guid`. El requisito es que la tabla `todos` debe contener el valor de la columna `guid` como la clave foránea y no el valor `id`. Esto se puede lograr así:

```ruby
class User < ApplicationRecord
  has_many :todos, primary_key: :guid
end
```

Ahora, si ejecutamos `@todo= @user.todos.create` entonces el valor `user_id` del registro `@todo` será el valor de `guid` de `@user`.

#### 4.3.2.9: source 

La opción: `source` especifica el nombre de la asociación de origen para una asociación `has_many: through`. Sólo es necesario utilizar esta opción si el nombre de la asociación de origen no puede deducirse automáticamente del nombre de la asociación. 

#### 4.3.2.10: source\_type 

La opción: `source_type` especifica el tipo de asociación fuente para un `has_many: through` de la asociación que procede a través de una asociación polimórfica.

#### 4.3.2.11: through 

La opción: `through` especifica un modelo de combinación mediante el cual realizar la consulta. Las asociaciones `has_many: through` proveen una manera de implementar relaciones muchos-a-muchos, como se discutió anteriormente en esta guía. 

#### 4.3.2.12: validate 

Si establece la opción: `validate` en `false`, los objetos asociados no se validarán cada vez que guarde este objeto. De forma predeterminada, esto es `true` donde los objetos asociados se validarán cuando se guarde este objeto. 



### 4.3.3 Scope de has\_many 

Puede haber ocasiones en las que desee personalizar la consulta utilizada por `has_many`. Tales personalizaciones se pueden conseguir a través de un bloque de scope. Por ejemplo:

```ruby
class Author < ApplicationRecord
  has_many :books, -> { where processed: true }
end
```

Puede utilizar cualquiera de los métodos de consulta estándar dentro del bloque de scope. A continuación se analizan las siguientes:

```ruby
where
extending
group
includes
limit
offset
order
readonly
select
distinct
```

#### 4.3.3.1 where 

El método `where` permite especificar las condiciones que debe cumplir el objeto asociado.

```ruby
class Author < ApplicationRecord
  has_many :confirmed_books, -> { where "confirmed = 1" },
    class_name: "Book"
end
```

También puede establecer las condiciones mediante un hash:

```ruby
class Author < ApplicationRecord
  has_many :confirmed_books, -> { where confirmed: true },
                              class_name: "Book"
end
```

  
Si utiliza una opción hash-style `where`, entonces la creación de registros a través de esta asociación se escaneará automáticamente utilizando el hash. En este caso, el uso de `@author.confirmed_books.create` o `@author.confirmed_books.build` creará libros donde la columna confirmada tiene el valor `true`.

#### 4.3.3.2 extending 

El método `extending` especifica un módulo con nombre para extender el proxy de asociación. Las extensiones de la asociación se discuten en detalle más adelante en esta guía. 

#### 4.3.3.3 group 

El método `group` proporciona un nombre de atributo para agrupar el conjunto de resultados, utilizando una cláusula `GROUP BY` en el buscador `SQL`.

```ruby
class Author < ApplicationRecord
  has_many :line_items, -> { group 'books.id' },
                        through: :books
end
```

#### 4.3.3.4 includes 

Puede utilizar el método `includes` para especificar asociaciones de segundo orden que deben cargarse con impaciencia cuando se utiliza esta asociación. Por ejemplo, considere estos modelos:

```ruby
class Author < ApplicationRecord
  has_many :books
end
 
class Book < ApplicationRecord
  belongs_to :author
  has_many :line_items
end
 
class LineItem < ApplicationRecord
  belongs_to :book
end
```

Si con frecuencia recupera `line_items` directamente de los `Author` \(`@author.books.line_items`\), puede hacer que su código sea algo más eficiente al incluir elementos de línea en la asociación de autores a libros:

```ruby
class Author < ApplicationRecord
  has_many :books, -> { includes :line_items }
end
 
class Book < ApplicationRecord
  belongs_to :author
  has_many :line_items
end
 
class LineItem < ApplicationRecord
  belongs_to :book
end
```

#### 4.3.3.5 limit 

El método `limit` le permite restringir el número total de objetos que se obtendrán a través de una asociación.

```ruby
class Author < ApplicationRecord
  has_many :recent_books,
    -> { order('published_at desc').limit(100) },
    class_name: "Book",
end
```

#### 4.3.3.6 offset 

El método `offset` le permite especificar el desplazamiento inicial para buscar objetos a través de una asociación. Por ejemplo, `->{offset (11)}` saltará los primeros 11 registros.

#### 4.3.3.7 order 

El método `order` determina el orden en el que se recibirán los objetos asociados \(en la sintaxis utilizada por una cláusula `SQL ORDER BY`\).

```ruby
class Author < ApplicationRecord
  has_many :books, -> { order "date_confirmed DESC" }
end
```

#### 4.3.3.8 readonly 

Si utiliza el método `readonly`, los objetos asociados serán de sólo lectura cuando se recuperen a través de la asociación. 

#### 4.3.3.9 select 

El método `select` le permite anular la cláusula `SQL SELECT` que se utiliza para recuperar datos sobre los objetos asociados. De forma predeterminada, Rails recupera todas las columnas. Si especifica su propia selección, asegúrese de incluir la clave principal y las columnas de clave externa del modelo asociado. Si no lo hace, Rails lanzará un error.

#### 4.3.3.10 distinct 

Utilice el método `distinct` para mantener la colección libre de duplicados. Esto es útil en su mayoría con la opción: `through`.

```ruby
class Person < ApplicationRecord
  has_many :readings
  has_many :articles, through: :readings
end
 
person = Person.create(name: 'John')
article   = Article.create(name: 'a1')
person.articles << article
person.articles << article
person.articles.inspect # => [#<Article id: 5, name: "a1">, #<Article id: 5, name: "a1">]
Reading.all.inspect     # => [#<Reading id: 12, person_id: 5, article_id: 5>, #<Reading id: 13, person_id: 5, article_id: 5>]
```

En el caso anterior hay dos `readings` y `person.articles` saca a relucir a ambos, aunque estos registros apuntan al mismo artículo. 

Ahora vamos a setear `distinct`:

```ruby
class Person
  has_many :readings
  has_many :articles, -> { distinct }, through: :readings
end
 
person = Person.create(name: 'Honda')
article   = Article.create(name: 'a1')
person.articles << article
person.articles << article
person.articles.inspect # => [#<Article id: 7, name: "a1">]
Reading.all.inspect     # => [#<Reading id: 16, person_id: 7, article_id: 7>, #<Reading id: 17, person_id: 7, article_id: 7>]
```

En el caso anterior todavía hay dos lecturas. Sin embargo `person.articles` muestra sólo un artículo porque la colección carga sólo registros únicos. 

Si desea asegurarse de que, al insertar, todos los registros de la asociación persistente son distintos \(para que pueda estar seguro de que al inspeccionar la asociación que nunca encontrará registros duplicados\), debe agregar un índice único en La propia tabla. Por ejemplo, si tiene una tabla denominada `readings` y desea asegurarse de que los artículos sólo se pueden agregar a una persona una vez, podría agregar lo siguiente en una migración:

```ruby
add_index :readings, [:person_id, :article_id], unique: true
```

Una vez que tenga este índice único, intentar agregar el artículo a una persona dos veces disparará un error de `ActiveRecord::RecordNotUnique`

```ruby
person = Person.create(name: 'Honda')
article = Article.create(name: 'a1')
person.articles << article
person.articles << article # => ActiveRecord::RecordNotUnique
```

Tenga en cuenta que la comprobación de la singularidad con algo como `include?` Está sujeto a condiciones de carrera. No intente utilizar `include?` Para reforzar la distinción en una asociación. Por ejemplo, usando el ejemplo del artículo de arriba, el código siguiente sería racy porque varios usuarios podrían estar intentando esto al mismo tiempo:

```ruby
person.articles << article unless person.articles.include?(article)person.articles << article unless person.articles.include?(article)
```



###  4.3.4 ¿Cuándo se guardan los objetos? 

Cuando asigna un objeto a una asociación `has_many`, ese objeto se guarda automáticamente \(para actualizar su clave externa\). Si asigna varios objetos en una sentencia, todos ellos se guardan. 

Si alguno de estos registros falla debido a errores de validación, la instrucción de asignación devuelve `false` y la asignación en sí se cancela. 

Si el objeto principal \(el que declara la asociación `has_many`\) no se guarda \(es decir, `new_record?` Devuelve `true`\), los objetos secundarios no se guardan cuando se agregan. Todos los miembros no guardados de la asociación se guardarán automáticamente cuando se guarda el padre. 

Si desea asignar un objeto a una asociación `has_many` sin guardar el objeto, utilice el método `collection.build`.

