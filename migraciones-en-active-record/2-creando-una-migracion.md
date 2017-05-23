# 2. Creando una Migración

### 2.1 Creando una Migración Independiente

Las migraciones son grabadas como ficheros en el directorio `db/migrate`, una por una para cada clase `migration`. El nombre del fichero es de la forma `YYYYMMDDHHMMSS_create_products.rb`, que es como decir un instante del tiempo UTC para identificar la migración, seguida por un guión bajo, seguido de un nombre de migración. El nombre de la clase migración \(versión CamelCased\) debería emparejar con la parte posterior del nombre del fichero. Por ejemplo en el fichero `20080906120000_create_products.rb` se deberia definir la clase `CreateProducts` y en el fichero `20080906120001_add_details_to_products.rb` se debería definir `AddDetailsToProducts`. Rails utiliza esta marca de tiempo para determinar cual migración debería ejecutarse en su correspondiente orden, entonces si tu estás copiando desde otra aplicación o generas el fichero por ti mismo, se consciente de sus posiciones en el orden.

Por supuesto, calcular los instantes no es divertido, entonces Active Record provee un generador que hace esto por ti:

```ruby
$ bin/rails generate migration AddPartNumberToProducts
```

Esto creará una migración vacía pero con el nombre apropiado:

```ruby
class AddPartNumberToProducts < ActiveRecord::Migration
  def change
  end
end
```

Si el nombre de la migración es de la forma "AddXXXToYYY" o "RemoveXXXFromYYY" y es seguido por una lista de nombres de columnas y tipos, será creada una migración conteniendo las declaraciones apropiadas `add_column` y `remove_column`

```ruby
$ bin/rails generate migration AddPartNumberToProducts part_number:string
```

generará

```ruby
class AddPartNumberToProducts < ActiveRecord::Migration
  def change
    add_column :products, :part_number, :string
  end
end
```

De modo similar, puedes generar una migración para remover una columna desde la línea de comandos:

```ruby
$ bin/rails generate migration RemovePartNumberFromProducts part_number:string
```

generará

```ruby
class RemovePartNumberFromProducts < ActiveRecord::Migration
  def change
    remove_column :products, :part_number, :string
  end
end
```

Si el nombre de la migración es de la forma "CreateXXX" y es seguida por una lista de nombres de columnas y tipos entonces será generada una migración para crear la tabla XXX con las columnas listadas. Por ejemplo:

```ruby
$ bin/rails generate migration CreateProducts name:string part_number:string
```

generará

```ruby
class CreateProducts < ActiveRecord::Migration
  def change
    create_table :products do |t|
      t.string :name
      t.string :part_number
    end
  end
end
```

Como siempre, lo que ha sido recientemente generado es un punto de partida. Puedes añadir o remover lo que sea adecuado editando el fichero `db/migrate/YYYYMMDDHHMMSS_add_details_to_products.rb`. También, el generador acepta tipos de columna como `references`\(también disponible como `belongs_to`\). Como ejemplo:

```ruby
$ bin/rails generate migration AddUserRefToProducts user:references
```

generará

```ruby
class AddUserRefToProducts < ActiveRecord::Migration
  def change
    add_reference :products, :user, index: true
  end
end
```

Esta migración creará una columna `user_id` y un índice apropiado. Hay también un generador que produce tablas de intersección, si JoinTable es parte del nombre:

```ruby
$ bin/rails g migration CreateJoinTableCustomerProduct customer product
```

producirá la siguiente migración:

```ruby
class CreateJoinTableCustomerProduct < ActiveRecord::Migration
  def change
    create_join_table :customers, :products do |t|
      # t.index [:customer_id, :product_id]
      # t.index [:product_id, :customer_id]
    end
  end
end
```

### 

### 2.2 Generadores del Modelo

Los generadores de modelos y de andamiaje \(Scaffold\) crearán las migraciones apropiadas para añadir un nuevo modelo. Estas migraciones ya contendrán las instrucciones para la creación de la correspondiente tabla. Si le dices a Rails que columnas quieres, las declaraciones para añadir esas columnas serán también creadas. Por ejemplo, ejecutando:

```ruby
$ bin/rails generate model Product name:string description:text
```

creará una migración que se ve de la siguiente manera:

```ruby
class CreateProducts < ActiveRecord::Migration
  def change
    create_table :products do |t|
      t.string :name
      t.text :description
 
      t.timestamps null: false
    end
  end
end
```

Puedes añadir la cantidad de columnas "nombre/tipo" que quieras.



### 2.3 Modificadores de Paso

Algo comúnmente utilizado es un tipo de modificador que puede ser pasados directamente en la línea de comandos. Están encerrados por llaves y después por el tipo de campo: Por ejemplo, ejecutando:

```ruby
$ bin/rails generate migration AddDetailsToProducts 'price:decimal{5,2}' supplier:references{polymorphic}
```

producirá una migración que luce como ésta:

```ruby
class AddDetailsToProducts < ActiveRecord::Migration
  def change
    add_column :products, :price, :decimal, precision: 5, scale: 2
    add_reference :products, :supplier, polymorphic: true, index: true
  end
end
```

Echa un vistazo a las ayudas de los generadores para mayor detalle.







