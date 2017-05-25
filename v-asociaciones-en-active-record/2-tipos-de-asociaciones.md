# 2. Tipos de Asociaciones

Rails soporta seis tipos de asociaciones:

```ruby
belongs_to
has_one
has_many
has_many :through
has_one :through
has_and_belongs_to_many
```

Las asociaciones se implementan mediante llamadas de macro-estilo, de modo que puede añadir declarativamente características a sus modelos. Por ejemplo, al declarar que un modelo pertenece a otro, instruye a Rails para que mantenga la información de la Primary Key - Foreign Key entre instancias de los dos modelos, y también obtiene una serie de métodos de utilidad agregados a su modelo. 

En el resto de esta guía, aprenderá a declarar y utilizar las diversas formas de asociaciones. Pero primero, una introducción rápida a las situaciones donde cada tipo de asociación es apropiado.



### 2.1 La asociación belongs\_to

Una asociación `belongs_to` establece una conexión uno a uno con otro modelo, de modo que cada instancia del modelo declarante "pertenece a" una instancia del otro modelo. Por ejemplo, si su aplicación incluye autores y libros, y cada libro se puede asignar a exactamente un autor, declararía el modelo de libro de esta manera:

```ruby
class Book < ApplicationRecord
  belongs_to :author
end
```

Las asociaciones `belongs_to` deben utilizar el término singular. Si usa el formulario pluralizado en el ejemplo anterior para la asociación de autores en el modelo de Book, se le diría que hay una constante no inicializada `uninitialized constant Book::Authors`. Esto se debe a que Rails infiere automáticamente el nombre de la clase del nombre de la asociación. Si el nombre de la asociación se pluraliza erróneamente, entonces la clase inferida también se pluralizará erróneamente. 

La migración correspondiente podría tener este aspecto:

```ruby
class CreateBooks < ActiveRecord::Migration[5.0]
  def change
    create_table :authors do |t|
      t.string :name
      t.timestamps
    end
    create_table :books do |t|
      t.belongs_to :author, index: true
      t.datetime :published_at
      t.timestamps
    end
  end
end
```



### 2.2 La asociación has\_one

Una asociación `has_one` también establece una conexión uno a uno con otro modelo, pero con una semántica algo diferente \(y consecuencias diferentes\). Esta asociación indica que cada instancia de un modelo contiene o posee una instancia de otro modelo. Por ejemplo, si cada proveedor de su aplicación tiene una sola cuenta, declararía el modelo de proveedor de la siguiente manera:

```ruby
class Supplier < ApplicationRecord
  has_one :account
end
```

La migración correspondiente podría tener este aspecto:

```ruby
class CreateSuppliers < ActiveRecord::Migration[5.0]
  def change
    create_table :suppliers do |t|
      t.string :name
      t.timestamps
    end
 
    create_table :accounts do |t|
      t.belongs_to :supplier, index: true
      t.string :account_number
      t.timestamps
    end
  end
end
```

Dependiendo del caso de uso, también puede ser necesario crear un índice único y / o una restricción de clave externa en la columna proveedor para la tabla de cuentas. En este caso, la definición de columna podría tener este aspecto:

```ruby
create_table :accounts do |t|
  t.belongs_to :supplier, index: { unique: true }, foreign_key: true
  # ...
end
```



### 2.3 La asociación has\_many

Una asociación `has_many` indica una conexión uno-a-muchos con otro modelo. A menudo encontrará esta asociación en el "otro lado" de una asociación `belongs_to`. Esta asociación indica que cada instancia del modelo tiene cero o más instancias de otro modelo. Por ejemplo, en una aplicación que contiene autores y libros, el modelo de autor podría ser declarado así:

```ruby
class Author < ApplicationRecord
  has_many :books
end
```

El nombre del otro modelo se pluraliza cuando se declara una asociación `has_many`. 

La migración correspondiente podría tener este aspecto:

```ruby
class CreateAuthors < ActiveRecord::Migration[5.0]
  def change
    create_table :authors do |t|
      t.string :name
      t.timestamps
    end
    create_table :books do |t|
      t.belongs_to :author, index: true
      t.datetime :published_at
      t.timestamps
    end
  end
end
```



### 2.4 La asociación has\_many :through

La asociación `has_many :through` se utiliza a menudo para establecer una conexión muchos-a-muchos con otro modelo. Esta asociación indica que el modelo de declaración puede coincidir con cero o más instancias de otro modelo, procediendo a través de un tercer modelo. Por ejemplo, considere una práctica médica donde los pacientes hacen citas para ver a los médicos. Las declaraciones de asociación pertinentes podrían tener este aspecto:

```ruby
class Physician < ApplicationRecord
  has_many :appointments
  has_many :patients, through: :appointments
end
 
class Appointment < ApplicationRecord
  belongs_to :physician
  belongs_to :patient
end
 
class Patient < ApplicationRecord
  has_many :appointments
  has_many :physicians, through: :appointments
end
```

La migración correspondiente podría tener este aspecto:

```ruby
class CreateAppointments < ActiveRecord::Migration[5.0]
  def change
    create_table :physicians do |t|
      t.string :name
      t.timestamps
    end
 
    create_table :patients do |t|
      t.string :name
      t.timestamps
    end
 
    create_table :appointments do |t|
      t.belongs_to :physician, index: true
      t.belongs_to :patient, index: true
      t.datetime :appointment_date
      t.timestamps
    end
  end
end
```

La colección de modelos de unión \(join\) se puede gestionar a través de los métodos de asociación `has_many`. Por ejemplo, si asigna:

```ruby
physician.patients = patients
```

A continuación, los nuevos modelos "Join" se crean automáticamente para los objetos recién asociados. Si algunas de las que ya existían anteriormente faltan, sus filas "Join" se eliminan automáticamente. 

La eliminación automática de los modelos de combinación es directa, no destruye los callbacks disparados 

La asociación `has_many :through` también es útil para configurar "accesos directos - shortcuts" a través de asociaciones anidadas \(nested\) `has_many`. Por ejemplo, si un documento tiene muchas secciones y una sección tiene muchos párrafos, a veces puede que desee obtener una colección simple de todos los párrafos del documento. Usted podría configurarlo de esta manera:

```ruby
class Document < ApplicationRecord
  has_many :sections
  has_many :paragraphs, through: :sections
end
 
class Section < ApplicationRecord
  belongs_to :document
  has_many :paragraphs
end

class Paragraph < ApplicationRecord
  belongs_to :section
end
```

Con `through:` especificando `:sections`, Rails ahora entenderá:

```ruby
@document.paragraphs
```



### 2.5 La asociación has\_one :through

Una asociación `has_one :through` crea una conexión uno a uno con otro modelo. Esta asociación indica que el modelo de declaración puede coincidir con una instancia de otro modelo procediendo a través de un tercer modelo. Por ejemplo, si cada proveedor tiene una cuenta y cada cuenta está asociada con un historial de cuenta, el modelo de proveedor podría tener este aspecto:

```ruby
class Supplier < ApplicationRecord
  has_one :account
  has_one :account_history, through: :account
end
 
class Account < ApplicationRecord
  belongs_to :supplier
  has_one :account_history
end
 
class AccountHistory < ApplicationRecord
  belongs_to :account
end
```

La migración correspondiente podría tener este aspecto:

```ruby
class CreateAccountHistories < ActiveRecord::Migration[5.0]
  def change
    create_table :suppliers do |t|
      t.string :name
      t.timestamps
    end
 
    create_table :accounts do |t|
      t.belongs_to :supplier, index: true
      t.string :account_number
      t.timestamps
    end
 
    create_table :account_histories do |t|
      t.belongs_to :account, index: true
      t.integer :credit_rating
      t.timestamps
    end
  end
end
```



### 2.6 La asociación has\_and\_belongs\_to\_many

Una asociación `has_and_belongs_to_many` crea una conexión directa many-to-many con otro modelo, sin modelo intermedio. Por ejemplo, si su aplicación incluye ensamblajes y partes, con cada ensamblaje teniendo muchas partes y cada parte apareciendo en muchos ensamblajes, podría declarar los modelos de esta manera:

```ruby
class Assembly < ApplicationRecord
  has_and_belongs_to_many :parts
end
 
class Part < ApplicationRecord
  has_and_belongs_to_many :assemblies
end
```

La migración correspondiente podría tener este aspecto:

```ruby
class CreateAssembliesAndParts < ActiveRecord::Migration[5.0]
  def change
    create_table :assemblies do |t|
      t.string :name
      t.timestamps
    end
    create_table :parts do |t|
      t.string :part_number
      t.timestamps
    end
 
    create_table :assemblies_parts, id: false do |t|
      t.belongs_to :assembly, index: true
      t.belongs_to :part, index: true
    end
  end
end
```

### 

### 2.7 Elegir entre belongs\_to y has\_one

Si desea establecer una relación de uno a uno entre dos modelos, deberá agregar `belongs_to` a uno y `has_one` al otro. ¿Cómo sabes cuál es cuál? 

La distinción está en donde se coloca la clave foránea \(va en la tabla para la clase que declara la asociación `belongs_to`\), pero debe reflexionar también sobre el significado real de los datos. La relación `has_one` dice que uno de algo es tuyo, es decir, que algo te indica algo. Por ejemplo, tiene más sentido decir que un proveedor posee una cuenta que una cuenta posee un proveedor. Esto sugiere que las relaciones correctas son así:

```ruby
class Supplier < ApplicationRecord
  has_one :account
end
 
class Account < ApplicationRecord
  belongs_to :supplier
end
```

La migración correspondiente podría tener este aspecto:

```ruby
class CreateSuppliers < ActiveRecord::Migration[5.0]
  def change
    create_table :suppliers do |t|
      t.string :name
      t.timestamps
    end
    create_table :accounts do |t|
      t.integer :supplier_id
      t.string  :account_number
      t.timestamps
    end
 
    add_index :accounts, :supplier_id
  end
end
```

El uso de `t.integer` `:supplier_id` hace que el nombre de clave externa sea obvio y explícito. En las versiones actuales de Rails, puede abstraer este detalle de implementación utilizando `t.references` `:supplier`.



### 2.8 Elegir entre has\_many: through y has\_and\_belongs\_to\_many

Rails ofrece dos formas diferentes de declarar una relación de muchos a muchos entre modelos. La forma más sencilla es usar `has_and_belongs_to_many`, lo que le permite hacer la asociación directamente:

```ruby
class Assembly < ApplicationRecord
  has_and_belongs_to_many :parts
end
 
class Part < ApplicationRecord
  has_and_belongs_to_many :assemblies
end
```

La segunda forma de declarar una relación muchos-a-muchos es usar `has_many :through`. Esto hace la asociación indirectamente, a través de un modelo de unión:

```ruby
class Assembly < ApplicationRecord
  has_many :manifests
  has_many :parts, through: :manifests
end
 
class Manifest < ApplicationRecord
  belongs_to :assembly
  belongs_to :part
end

class Part < ApplicationRecord
  has_many :manifests
  has_many :assemblies, through: :manifests
end
```

La regla más sencilla es que debe establecer una relación `has_many :through` si necesita trabajar con el modelo de relación como una entidad independiente. Si no necesita hacer nada con el modelo de relación, puede ser más sencillo configurar una relación `has_and_belongs_to_many` \(aunque deberá recordar crear la "join table" en la base de datos\). 

Debe utilizar `has_many: :through` si necesita validaciones, callbacks o atributos adicionales en el modelo de union.



### 2.9 Asociaciones polimórficas

Un giro un poco más avanzado en las asociaciones es la asociación polimórfica. Con asociaciones polimórficas, un modelo puede pertenecer a más de un modelo, en una sola asociación. Por ejemplo, es posible que tenga un modelo "imagen" que pertenezca a un modelo de empleado o un modelo de producto. He aquí cómo se podría declarar esto:

```ruby
class Picture < ApplicationRecord
  belongs_to :imageable, polymorphic: true
end
 
class Employee < ApplicationRecord
  has_many :pictures, as: :imageable
end
 
class Product < ApplicationRecord
  has_many :pictures, as: :imageable
end
```

Puede pensar en una declaración polimórfica de `belongs_to` como configurar una interfaz que cualquier otro modelo puede usar. Desde una instancia del modelo `Employee`, puede recuperar una colección de imágenes: `@employee.pictures`. 

Del mismo modo, puede recuperar `@product.pictures`. 

Si tiene una instancia del modelo de imagen, puede llegar a su padre a través de `@picture.imageable`. Para que esto funcione, debe declarar tanto una columna de clave foranea como una columna "type" en el modelo que declare la interfaz polimórfica:

```ruby
class CreatePictures < ActiveRecord::Migration[5.0]
  def change
    create_table :pictures do |t|
      t.string  :name
      t.integer :imageable_id
      t.string  :imageable_type
      t.timestamps
    end
 
    add_index :pictures, [:imageable_type, :imageable_id]
  end
end
```

Esta migración puede simplificarse mediante el uso de la forma `t.references`:

```ruby
class CreatePictures < ActiveRecord::Migration[5.0]
  def change
    create_table :pictures do |t|
      t.string :name
      t.references :imageable, polymorphic: true, index: true
      t.timestamps
    end
  end
end
```



### 2.10 Self Joins

En el diseño de un modelo de datos, a veces se encuentra un modelo que debe tener una relación con sí mismo. Por ejemplo, es posible que desee almacenar todos los empleados en un único modelo de base de datos, pero pueda rastrear las relaciones, como por ejemplo entre el administrador y los subordinados. Esta situación puede ser modelada con asociaciones Self Joins:

```ruby
class Employee < ApplicationRecord
  has_many :subordinates, class_name: "Employee",
                          foreign_key: "manager_id"
 
  belongs_to :manager, class_name: "Employee"
end
```

Con esta configuración, puede recuperar `@employee.subordinates` y `@employee.manager`.

En sus migraciones / esquema, agregará una columna de referencias al propio modelo.

```ruby
class CreateEmployees < ActiveRecord::Migration[5.0]
  def change
    create_table :employees do |t|
      t.references :manager, index: true
      t.timestamps
    end
  end
end
```







