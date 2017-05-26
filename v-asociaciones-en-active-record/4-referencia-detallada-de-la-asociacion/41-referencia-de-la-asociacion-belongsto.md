# 4.1 Referencia de la asociación belongs\_to

La asociación belongs\_to crea una coincidencia uno-a-uno con otro modelo. En términos de base de datos, esta asociación dice que esta clase contiene la clave foránea. Si la otra clase contiene la clave foránea, entonces debería usar `has_one` en su lugar.

### 4.1.1 Métodos agregados por belongs\_to

Cuando declara una asociación `belongs_to`, la clase declarante obtiene automáticamente cinco métodos relacionados con la asociación:

```ruby
association
association=(associate)
build_association(attributes = {})
create_association(attributes = {})
create_association!(attributes = {})
```

En todos estos métodos, la palabra `association` se sustituye por el símbolo pasado como el primer argumento a `belongs_to`. Por ejemplo, dada la declaración:

```ruby
class Book < ApplicationRecord
  belongs_to :author
end
```

Cada instancia del modelo `Book` tendrá estos métodos:

```ruby
Book.author
Book.author=
Book.build_author
Book.create_author
Book.create_author!
```

Al inicializar una nueva asociación `has_one` o `belongs_to`, debe utilizar el prefijo `build_` para crear la asociación, en lugar del método `association.build` que se utilizaría para las asociaciones `has_many` o `has_and_belongs_to_many`. Para crear uno, use el prefijo `create_`.

#### 4.1.1.1 association

El método de `association` devuelve el objeto asociado, si lo hay. Si no se encuentra ningún objeto asociado, devuelve nil.

```ruby
@author = @book.author
```

Si el objeto asociado ya se ha recuperado de la base de datos para este objeto, se devolverá la versión almacenada en caché. Para anular este comportamiento \(y forzar una base de datos no leída\), llame `#reload` en el objeto primario.

```ruby
@author = @book.reload.author
```

#### 4.1.1.2 association=\(associate\)

El método `association=` asigna un objeto asociado a este objeto. Detrás de las escenas, esto significa extraer la clave principal del objeto asociado y establecer la clave externa de este objeto en el mismo valor.

```ruby
@book.author = @author
```

#### 4.1.1.3 build\_association\(attributes = {}\)

El método `build_association` devuelve un nuevo objeto del tipo asociado. Este objeto se instanciará a partir de los atributos pasados y se establecerá el vínculo a través de la clave externa de este objeto, pero el objeto asociado aún no se guardará.

```ruby
@author = @book.build_author(author_number: 123,
                                  author_name: "John Doe")
```

#### 4.1.1.4 create\_association\(attributes = {}\)

El método `create_association` devuelve un nuevo objeto del tipo asociado. Este objeto será instanciado a partir de los atributos pasados, se establecerá el enlace a través de la clave externa de este objeto y, una vez que pase todas las validaciones especificadas en el modelo asociado, se guardará el objeto asociado.

```ruby
@author = @book.create_author(author_number: 123,
                                   author_name: "John Doe")
```

#### 4.1.1.5 create\_association!\(attributes = {}\)

Hace lo mismo que `create_association`, pero activa `ActiveRecord::RecordInvalid` si el registro no es válido.

### 4.1.2 Opciones para belongs\_to

Mientras que Rails utiliza valores predeterminados inteligentes que funcionarán bien en la mayoría de las situaciones, puede haber ocasiones en las que desee personalizar el comportamiento de la referencia de asociación `belongs_to`. Tales personalizaciones pueden realizarse fácilmente al pasar opciones y bloques de ámbito al crear la asociación. Por ejemplo, esta asociación utiliza dos opciones:

```ruby
class Book < ApplicationRecord
  belongs_to :author, dependent: :destroy,
    counter_cache: true
end
```

La asociación belongs\_to soporta estas opciones:

```ruby
:autosave
:class_name
:counter_cache
:dependent
:foreign_key
:primary_key
:inverse_of
:polymorphic
:touch
:validate
:optional
```

#### 4.1.2.1: autosave

Si configura la opción `:autosave` en `true`, Rails guardará todos los miembros cargados y destruirá los miembros marcados para su destrucción cada vez que guarde el objeto principal.

#### 4.1.2.2: class\_name

Si el nombre del otro modelo no puede derivarse del nombre de la asociación, puede utilizar la opción: `class_name` para proporcionar el nombre del modelo. Por ejemplo, si un libro pertenece a un autor, pero el nombre real del modelo que contiene los autores es `Patron`, se establecerían las cosas de esta manera:

```ruby
class Book < ApplicationRecord
  belongs_to :author, class_name: "Patron"
end
```

#### 4.1.2.3: counter\_cache

La opción: `counter_cache` se puede utilizar para hacer más eficiente el hallazgo del número de objetos pertenecientes. Considere estos modelos:

```ruby
class Book < ApplicationRecord
  belongs_to :author
end
class Author < ApplicationRecord
  has_many :books
end
```

Con estas declaraciones, pedir el valor de `@author.books.size` requiere realizar una llamada a la base de datos para realizar una consulta `COUNT(*)`. Para evitar esta llamada, puede agregar una caché de contador al modelo de pertenencia:

```ruby
class Book < ApplicationRecord
  belongs_to :author, counter_cache: true
end
class Author < ApplicationRecord
  has_many :books
end
```

Con esta declaración, Rails mantendrá el valor de caché actualizado y, a continuación, devolverá ese valor en respuesta al método `.size`.

Aunque la opción: `counter_cache` se especifica en el modelo que incluye la declaración `belongs_to`, la columna real se debe agregar al modelo asociado \(`has_many`\). En el caso anterior, debería agregar una columna denominada `books_count` al modelo `Author`.

Puede reemplazar el nombre de columna predeterminado especificando un nombre de columna personalizado en la declaración `counter_cache` en lugar de `true`. Por ejemplo, para usar `count_of_books` en lugar de `books_count`:

```ruby
class Book < ApplicationRecord
  belongs_to :author, counter_cache: :count_of_books
end
class Author < ApplicationRecord
  has_many :books
end
```

Sólo es necesario especificar la opción: `counter_cache` en el lado `belongs_to` de la asociación.

Las columnas de Counter Cache se agregan a la lista de atributos de solo lectura de modelo de contenedor a través de `attr_readonly`.

#### 4.1.2.4 :dependent

Controla qué sucede con los objetos asociados cuando se destruye a su propietario:

* `:destroy` hace que los objetos asociados también se destruyan. 
* `:delete_all` hace que los objetos asociados sean eliminados directamente de la base de datos \(los callbacks no se ejecutan\). 
* `:nullify` hace que las claves foráneas se establezcan en NULL \(callbacks no se ejecutan\). 
* `:restrict_with_exception` provoca que se genere una excepción si hay registros asociados. 
* `:restrict_with_error` causa que se agregue un error al propietario si hay objetos asociados.

No debe especificar esta opción en una asociación `belongs_to` que está conectada con una asociación `has_many` en la otra clase. Hacerlo puede llevar a registros huérfanos en su base de datos.

#### 4.1.2.5 :foreign\_key

Por convención, Rails asume que la columna utilizada para mantener la clave foránea en este modelo es el nombre de la asociación con el sufijo `_id` añadido. La opción: `foreign_key` le permite establecer el nombre de la clave externa directamente:

```ruby
class Book < ApplicationRecord
  belongs_to :author, class_name: "Patron",
                        foreign_key: "patron_id"
end
```

En cualquier caso, Rails no creará columnas de clave foranea para usted. Debe definirlos explícitamente como parte de sus migraciones.

#### 4.1.2.6: primary\_key

Por convención, Rails asume que la columna `id` se utiliza para contener la clave primaria de sus tablas. La opción: `primary_key` le permite especificar una columna diferente.

Por ejemplo, dado que tenemos una tabla de usuarios con `guid` como la clave principal. Si queremos una tabla separada de todos para mantener la clave externa `user_id` en la columna `guid`, entonces podemos usar `primary_key` para lograr esto:

```ruby
class User < ApplicationRecord
  self.primary_key = 'guid' # primary key is guid and not id
end

class Todo < ApplicationRecord
  belongs_to :user, primary_key: 'guid'
end
```

Cuando ejecutamos `@user.todos.create` entonces el registro `@todo` tendrá su valor `user_id` como el valor de `guid` de `@user`.

#### 4.1.2.7 :inverse\_of

La opción `:inverse_of` especifica el nombre de la asociación `has_many` o `has_one` que es la inversa de esta asociación. No funciona en combinación con la opcion `:polymorphic`.

```ruby
class Author < ApplicationRecord
  has_many :books, inverse_of: :author
end

class Book < ApplicationRecord
  belongs_to :author, inverse_of: :books
end
```

#### 4.1.2.8 :polymorphic

Pasar `true` a la opción `:polymorphic` indica que se trata de una asociación polimórfica. Las asociaciones polimórficas se discutieron en detalle anteriormente en esta guía.

#### 4.1.2.9 :touch

Si establece la opción: `touch` en `true`, entonces la marca de tiempo `updated_at` o `updated_on` en el objeto asociado se establecerá en la hora actual siempre que se guarde o destruya este objeto:

```ruby
class Book < ApplicationRecord
  belongs_to :author, touch: true
end

class Author < ApplicationRecord
  has_many :books
end
```

En este caso, guardar o destruir un libro actualizará la marca de tiempo en el autor asociado. También puede especificar un atributo de marca de tiempo específico para actualizar:

```ruby
class Book < ApplicationRecord
  belongs_to :author, touch: :books_updated_at
end
```

#### 4.1.2.10 :validate

Si establece la opción `:validate` en `true`, los objetos asociados se validarán cada vez que guarde este objeto. De forma predeterminada es `false.` los objetos asociados no se validarán cuando se guarde este objeto.

#### 4.1.2.11 :optional

Si establece la opción `:optional` en `true`, entonces la presencia del objeto asociado no será validada. De forma predeterminada, esta opción se establece en `false`.

### 4.1.3 Scope de belongs\_to

Puede haber ocasiones en las que desee personalizar la consulta utilizada por `belongs_to`. Tales personalizaciones se pueden conseguir a través de un bloque de Scope. Por ejemplo:

```ruby
class Book < ApplicationRecord
  belongs_to :author, -> { where active: true },
                        dependent: :destroy
end
```

Puede utilizar cualquiera de los métodos de consulta estándar dentro del bloque de Scope. A continuación se analizan las siguientes:

```ruby
where
includes
readonly
select
```

#### 4.1.3.1 where

El método `where` permite especificar las condiciones que debe cumplir el objeto asociado.

```ruby
class book < ApplicationRecord
  belongs_to :author, -> { where active: true }
end
```

#### 4.1.3.2 include

Puede utilizar el método `include` para especificar asociaciones de segundo orden que deben cargarse con impaciencia cuando se utiliza esta asociación. Por ejemplo, considere estos modelos:

```ruby
class LineItem < ApplicationRecord
  belongs_to :book
end

class Book < ApplicationRecord
  belongs_to :author
  has_many :line_items
end

class Author < ApplicationRecord
  has_many :books
end
```

Si con frecuencia recupera autores directamente de `line_item`\(`@line_item.book.author`\), puede hacer que su código sea un poco más eficiente al incluir a los autores en la asociación de `line_item` a `books`:

```ruby
class LineItem < ApplicationRecord
  belongs_to :book, -> { includes :author }
end

class Book < ApplicationRecord
  belongs_to :author
  has_many :line_items
end

class Author < ApplicationRecord
  has_many :books
end
```

No hay necesidad de utilizar `include` para asociaciones inmediatas, es decir, si tienes `Book belongs_to: author`, entonces el autor se carga con impaciencia automáticamente cuando es necesario.

#### 4.1.3.3 readonly

Si utiliza readonly, el objeto asociado será de sólo lectura cuando se recupera a través de la asociación.

#### 4.1.3.4 select 

El método `select` le permite anular la cláusula `SQL SELECT` que se utiliza para recuperar datos sobre el objeto asociado. De forma predeterminada, Rails recupera todas las columnas. Si utiliza el método `select` en una asociación `belongs_to`, también debe establecer la opción: `foreign_key` para garantizar los resultados correctos.



### 4.1.4 ¿Existen objetos asociados? 

Puede ver si existen objetos asociados usando el metodo `association.nil?`:

```ruby
if @book.author.nil?
  @msg = "No author found for this book"
end
```



### 4.1.5 ¿Cuándo se guardan los objetos? 

Asignar un objeto a una asociación `belongs_to` no guarda automáticamente el objeto. Tampoco guarda el objeto asociado.

