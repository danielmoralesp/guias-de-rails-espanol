### 3.  Consejos, Trucos y Advertencias

Estas son algunas de las cosas que debe saber para hacer un uso eficiente de las asociaciones Active Record en sus aplicaciones de Rails:

* Control del almacenamiento en caché 
* Evitar colisiones de nombres 
* Actualización del esquema 
* Controlar el alcance de la asociación 
* Asociaciones bidireccionales

### 3.1 Control del almacenamiento en caché

Todos los métodos de asociación se construyen alrededor del almacenamiento en caché, lo que mantiene el resultado de la consulta más reciente disponible para operaciones adicionales. La caché se comparte incluso entre los métodos. Por ejemplo:

```ruby
author.books                 # recupera los libros de la base de datos
author.books.size            # utiliza la copia en caché de libros
author.books.empty?          # utiliza la copia en caché de libros
```

Pero ¿qué pasa si desea volver a cargar la caché, porque los datos podrían haber sido cambiados por alguna otra parte de la aplicación? Simplemente llame a "reload" en la asociación:

```ruby
author.books                 # recupera los libros de la base de datos
author.books.size            # utiliza la copia en caché de libros
author.books.reload.empty?   # Descarta la copia en caché de libros
                              # Y vuelve a la base de datos
```

### 3.2 Evitar las colisiones de nombres

No eres libre de usar cualquier nombre para tus asociaciones. Debido a que crear una asociación agrega un método con ese nombre al modelo, es una mala idea darle a una asociación un nombre que ya está utilizado para un método de instancia de `ActiveRecord::Base`. El método de asociación invalidaría el método base y rompería las cosas. Por ejemplo, `attributes` o `connection` son malos nombres para las asociaciones.



### 3.3 Actualización del esquema

Las asociaciones son extremadamente útiles, pero no son mágicas. Usted es responsable de mantener el esquema de la base de datos para que coincida con sus asociaciones. En la práctica, esto significa dos cosas, dependiendo del tipo de asociaciones que está creando. Para asociaciones `belongs_to` es necesario crear claves foráneas y para asociaciones `has_and_belongs_to_many` es necesario crear la tabla de unión \(join table\) adecuada.

#### 3.3.1 Creación de claves foráneas para las asociaciones belongs\_to 

Cuando declara una asociación `belongs_to`, debe crear claves foráneas según corresponda. Por ejemplo, considere este modelo:

```ruby
class Book < ApplicationRecord
  belongs_to :author
end
```

Esta declaración debe ser respaldada por la declaración de clave foránea adecuada en la tabla de libros:

```ruby
class CreateBooks < ActiveRecord::Migration[5.0]
  def change
    create_table :books do |t|
      t.datetime :published_at
      t.string   :book_number
      t.integer  :author_id
    end
  end
end
```

Si crea una asociación algún tiempo después de crear el modelo subyacente, debe recordar crear una migración `add_column` para proporcionar la clave foránea necesaria. 

Es una buena práctica agregar un índice en la clave foránea para mejorar el rendimiento de las consultas y una restricción de clave foránea para garantizar la integridad de los datos referenciales:

```ruby
class CreateBooks < ActiveRecord::Migration[5.0]
  def change
    create_table :books do |t|
      t.datetime :published_at
      t.string   :book_number
      t.integer  :author_id
    end
 
    add_index :books, :author_id
    add_foreign_key :books, :authors
  end
end
```

#### 3.3.2 Creación de tablas de unión \(join tables\) para las asociaciones has\_and\_belongs\_to\_many 

Si crea una asociación `has_and_belongs_to_many`, debe crear explícitamente la tabla de unión. A menos que el nombre de la tabla de unión se especifique explícitamente utilizando la opción: `join_table`, Active Record crea el nombre utilizando el léxico de los nombres de clase. Por lo tanto, una combinación entre los modelos autor y libros dará el nombre de tabla de unión predeterminada de `authors_books` porque "a" supera a "b" en el orden alfabético.

La precedencia entre los nombres de modelo se calcula utilizando el operador `<=>` para String. Esto significa que si las cadenas son de diferentes longitudes y las cadenas son iguales en comparación con la longitud más corta, entonces la cadena más larga se considera de mayor precedencia léxica que la más corta. Por ejemplo, se podría esperar que las tablas `paper_boxes` y `papers` generen un nombre de tabla de unión de `papers_paper_boxes` debido a la longitud del nombre `paper_boxes`, pero de hecho genera un nombre de tabla de unión de `paper_boxes_papers` \( Porque el subrayado '\_' es lexicográficamente menor que 's' en codificaciones comunes\).

Independientemente del nombre, debe generar manualmente la tabla de unión con una migración adecuada. Por ejemplo, considere estas asociaciones:

```ruby
class Assembly < ApplicationRecord
  has_and_belongs_to_many :parts
end
 
class Part < ApplicationRecord
  has_and_belongs_to_many :assemblies
end
```

Estos deben ser respaldados por una migración para crear la tabla `assemblies_parts`. Esta tabla debe crearse sin una clave primaria:

```ruby
class CreateAssembliesPartsJoinTable < ActiveRecord::Migration[5.0]
  def change
    create_table :assemblies_parts, id: false do |t|
      t.integer :assembly_id
      t.integer :part_id
    end
 
    add_index :assemblies_parts, :assembly_id
    add_index :assemblies_parts, :part_id
  end
end
```

Pasamos `id :false` a `create_table` porque esa tabla no representa un modelo. Eso es necesario para que la asociación funcione correctamente. Si observas cualquier comportamiento extraño en una asociación `has_and_belongs_to_many` como ID de modelo mutilado, o excepciones sobre IDs en conflicto, es probable que te hayas olvidado de ese detalle.

También puede utilizar el método `create_join_table`

```ruby
class CreateAssembliesPartsJoinTable < ActiveRecord::Migration[5.0]
  def change
    create_join_table :assemblies, :parts do |t|
      t.index :assembly_id
      t.index :part_id
    end
  end
end
```



### 3.4 Control del alcance de la asociación

De forma predeterminada, las asociaciones buscan objetos sólo dentro del ámbito del módulo actual. Esto puede ser importante cuando declara los modelos Active Record dentro de un módulo. Por ejemplo:

```ruby
module MyApplication
  module Business
    class Supplier < ApplicationRecord
       has_one :account
    end
 
    class Account < ApplicationRecord
       belongs_to :supplier
    end
  end
end
```

Esto funcionará bien, ya que tanto la clase proveedor como la clase cuenta se definen dentro del mismo ámbito. Pero lo siguiente no funcionará, ya que el proveedor y la cuenta se definen en ámbitos diferentes:

```ruby
module MyApplication
  module Business
    class Supplier < ApplicationRecord
       has_one :account
    end
  end
  module Billing
    class Account < ApplicationRecord
       belongs_to :supplier
    end
  end
end
```

Para asociar un modelo con un modelo en un `namespace` diferente, debe especificar el nombre completo de la clase en la declaración de asociación:

```ruby
module MyApplication
  module Business
    class Supplier < ApplicationRecord
       has_one :account,
        class_name: "MyApplication::Billing::Account"
    end
  end
 
  module Billing
    class Account < ApplicationRecord
       belongs_to :supplier,
        class_name: "MyApplication::Business::Supplier"
    end
  end
end
```



### 3.5 Asociaciones bidireccionales

Es normal que las asociaciones trabajen en dos direcciones, requiriendo la declaración en dos modelos diferentes:

```ruby
class Author < ApplicationRecord
  has_many :books
end
 
class Book < ApplicationRecord
  belongs_to :author
end
```

Active Record intentará identificar automáticamente que estos dos modelos comparten una asociación bidireccional basada en el nombre de la asociación. De esta manera, Active Record sólo cargará una copia del objeto `Author`, haciendo que su aplicación sea más eficiente y evite datos incoherentes:

```ruby
a = Author.first
b = a.books.first
a.first_name == b.author.first_name # => true
a.first_name = 'David'
a.first_name == b.author.first_name # => true
```

Active Record admite la identificación automática para la mayoría de las asociaciones con nombres estándar. Sin embargo, Active Record no identificará automáticamente asociaciones bidireccionales que contengan cualquiera de las siguientes opciones:

```ruby
:conditions
:through
:polymorphic
:class_name
:foreign_key
```

Por ejemplo, considere las siguientes declaraciones de modelo:

```ruby
class Author < ApplicationRecord
  has_many :books
end
 
class Book < ApplicationRecord
  belongs_to :writer, class_name: 'Author', foreign_key: 'author_id'
end
```

Active Record ya no reconocerá automáticamente la asociación bidireccional:

```ruby
a = Author.first
b = a.books.first
a.first_name == b.writer.first_name # => true
a.first_name = 'David'
a.first_name == b.writer.first_name # => false
```

Active Record proporciona la opción: `inverse_of` para que pueda declarar explícitamente asociaciones bidireccionales:

```ruby
class Author < ApplicationRecord
  has_many :books, inverse_of: 'writer'
end
 
class Book < ApplicationRecord
  belongs_to :writer, class_name: 'Author', foreign_key: 'author_id'
end
```

Al incluir la opción: `inverse_of` en la declaración de asociación `has_many`, Active Record reconocerá ahora la asociación bidireccional:

```ruby
a = Author.first
b = a.books.first
a.first_name == b.writer.first_name # => true
a.first_name = 'David'
a.first_name == b.writer.first_name # => true
```

Existen algunas limitaciones con `:inverse_of` 

* No trabajan con asociaciones `:through`. 
* No funcionan con asociaciones polimórficas. 
* No trabajan con asociaciones `:as`



