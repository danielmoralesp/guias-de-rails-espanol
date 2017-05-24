# 3. Escribiendo una Migración

¡Una vez que has creado tu migración utilizando uno de los generadores es tiempo de comenzar el trabajo!

### 3.1 Creando una Tabla

El método `create_table` es uno de los fundamentales, pero la mayoría del tiempo, los generarás utilizando el generador de modelo o de andamiaje \(Scaffold\). Un típico uso podría ser:

```ruby
create_table :products do |t|
  t.string :name
end
```

El cual crea una tabla `products` con una columna llamada `name` \(y como se discutirá a continuación, una columna implícita `id`\).

Por defecto, `create_table` creará una clave primaria llamada `id`. Puedes cambiar el nombre de la clave primaria con la opción `:primary_key` \(no te olvides de actualizar el modelo correspondiente\) o, si no quieres ninguna clave primaria, puedes escribir la opción `id: false`. Si necesitas pasarle a la base de datos opciones específicas puedes escribir un fragmento de SQĹ en la opción `:options`. Por ejemplo:

```ruby
create_table :products, options: "ENGINE=BLACKHOLE" do |t|
  t.string :name, null: false
end
```

Esto añadirá `ENGINE=BLACKHOLE` con la declaración SQL utilizada para crear una tabla \(cuando utilizamos MySQL, por defecto es `ENGINE=InnoDB`\).

### 3.2 Crear una Tabla de Intersección

El método de migración `create_join_table` crea una tabla de intersección HABTM \(Has And Belongs To Many\). Una típica podría ser:

```ruby
create_join_table :products, :categories
```

La cual creará una tabla `categories_products` con dos columnas llamadas `category_id` y `product_id`. Esas columnas tienen la opción `:null` configurada a `false` por defecto. Esto será sobrescrito especificando la opción `:column_options`.

```ruby
create_join_table :products, :categories, column_options: {null: true}
```

Creará `product_id` y `category_id` con la opción `:null` a `true`.

Puedes pasar la opción `:table_name` cuando quieras para adaptar el nombre de la tabla a tus necesidades. Por ejemplo:

```ruby
create_join_table :products, :categories, table_name: :categorization
```

creará una tabla `categorization`.

`create_join_table` también acepta un bloque, el cual puedes utilizar para añadir índices \(que no son creados por defecto\) o columnas adicionales:

```ruby
create_join_table :products, :categories do |t|
  t.index :product_id
  t.index :category_id
end
```

### 

### 3.3 Cambiando Tablas

La prima hermana de `create_table` es `change_table`, utilizada para modificar tablas existentes. Esto es utilizado de forma similar a `create_table` pero el objeto que se cede al bloque conoce más trucos. Por ejemplo:

```ruby
change_table :products do |t|
  t.remove :description, :name
  t.string :part_number
  t.index :part_number
  t.rename :upccode, :upc_code
end
```

Al igual que `remove_column` y `add_column`, Rails proporciona el método de migración `change_column`.

```ruby
change_column :products, :part_number, :text
```

Esto cambia la columna `part_number` sobre la tabla `products` para convertirla en un campo de `texto`.

Además de `change_column`, los métodos `change_column_null` y `change_column_default` son utilizados específicamente para cambiar el nulo y el valor por defecto de una columna.

```ruby
change_column_null :products, :name, false
change_column_default :products, :approved, false
```

Esto establece el campo `:name` de `products` a una columna `NOT NULL` y el valor por defecto del campo `:approved` a `false`.

A diferencia de `change_column` \(y `change_column_default`\), `change_column_null` es reversible.

### 3.5 Modificadores de Columnas

Los modificadores de columnas pueden ser aplicados cuando se crea o se cambia una columna:

* `limit` Establece el máximo tamaño de un campo string/text/binary/integer. 
* `precision` Define la precisión de los campos decimal, representando el número de dígitos total del número. 
* `scale` Define la escala para los campos decimal, representando el numero de dígitos después del punto decimal. 
* `polymorphic` Añade una columna `type` para las asociaciones `belongs_to`. 
* `null` Añade o rechaza valores `NULL` en la columna. 
* `default` Permite establecer un valor por defecto de la columna. Nota que si utilizas un valor dinámico \(como una fecha\), el valor por defecto solo será calculado la primera vez \(ej: en la fecha y hora que la migración es aplicada\). 
* `index` Añade un índice para la columna. 
* `required` Añade `required: true` para las asociaciones `belongs_to` y `null: false` para la columna de la migración.

Algunos adaptadores pueden soportar opciones adicionales; ver el adaptador específico en los documentos API para más información.

### 3.6 Claves foráneas

Mientras que no es requerida podrías querer añadir una restricción de clave foránea para garantizar la integridad referencial.

```ruby
add_foreign_key :articles, :authors
```

Esto añade una nueva clave foránea a la columna `author_id` en la tabla `articles`. La clave hace referencia a la columna `id` de la tabla `authors`. Si los nombres de las columnas no pueden ser derivados de los nombres de las tablas, puedes utilizar ls opciones `:column` y `:primary_key`.

Rails generará un nombre para cada clave foránea empezando por `fk_rails_` seguido por 10 caracteres aleatorios. Hay una opción `:name` para especificar un nombre diferente si fuera necesario.

Active Record solo soporta una columna simple como clave foránea. `execute` y `structure.sql` son requeridas para utilizar claves foráneas compuestas.

Remover una clave foránea es tan fácil como:

```ruby
# decirle a Active Record que la averigue por los modelos involucrados
remove_foreign_key :accounts, :branches

# borrar la clave foranea de una columna específica
remove_foreign_key :accounts, column: :owner_id

# borrar la clave foranea por su nombre
```

### 

### 3.7 Cuando los Helpers no son Suficientes

Si los helpers provistos por Active Record no son suficientes puedes utilizar el método `execute` para ejecutar cualquier SQL:

```ruby
Product.connection.execute('UPDATE `products` SET `price`=`free` WHERE 1')
```

Para más detalles y ejemplos de métodos individuales, leer la documentación API. En particular la documentación para 

* `ActiveRecord::ConnectionAdapters::SchemaStatements` \(la cual provee los métodos disponibles en los métodos `change`, `up` y `down`\), 
* `ActiveRecord::ConnectionAdapters::TableDefinition` \(el cual provee los métodos disponibles en el objeto cedido por `create_table`\) y 
* `ActiveRecord::ConnectionAdapters::Table` \(el cual provee los métodos disponibles en el objeto cedido por `change_table`\).



### 3.8 Utilizando el Método change

El método `change` es la principal manera de escribir migraciones. Este funciona para la mayoría de los casos, donde Active Record conoce como revertir la migración automáticamente. Actualmente, el método `change` soporta solo estas definiciones de migración:

```ruby
add_column
add_index
add_reference
add_timestamps
add_foreign_key
create_table
create_join_table
drop_table (must supply a block)
drop_join_table (must supply a block)
remove_timestamps
rename_column
rename_index
remove_reference
rename_table
```

`change_table` es también reversible, siempre y cuando el bloque no llame a `change`, `change_default` o `remove`. Si necesitas utilizar algún otro método, deberías usar `reversible` o escribir los métodos `up` y `down` en lugar de utilizar el método `change`.



### 3.9 Usando reversible

Las migraciones complejas pueden requerir procesamiento que Active Record no sabe como revertir. Puedes utilizar `reversible` para especificar que hacer cuando estás ejecutando una migración y será lo que se haga cuando se revierta. Por ejemplo:

```ruby
class ExampleMigration < ActiveRecord::Migration
  def change
    create_table :distributors do |t|
      t.string :zipcode
    end
 
    reversible do |dir|
      dir.up do
        # añadir una restricción CHECK
        execute <<-SQL
          ALTER TABLE distributors
            ADD CONSTRAINT zipchk
              CHECK (char_length(zipcode) = 5) NO INHERIT;
        SQL
      end
      dir.down do
        execute <<-SQL
          ALTER TABLE distributors
            DROP CONSTRAINT zipchk
            SQL
      end
    end
 
    add_column :users, :home_page_url, :string
    rename_column :users, :email, :email_address
  end
end
```

Utilizando `reversible` nos aseguraremos que las instrucciones son ejecutadas en el orden correcto también. Si el ejemplo de la anterior migración es revertido, el bloque `down` será ejecutado después de que la columna `home_page_url` sea borrada y justo antes de que la tabla `distributors` se borre. 

Algunas veces tu migración hará algo que es simplemente irreversible, por ejemplo, puede borrar algunos datos, En tales casos, puedes lanzar una excepción `ActiveRecord::IrreversibleMigration` en tu bloque `down`. Si alguien intenta revertir tu migración, se le mostrará un mensaje de error diciendo que no se pude hacer.



### 3.10 Utilizando los Métodos up/down

También puedes utilizar los viejos métodos `up` y `down` en lugar del método `change`. El método `up` debería describir la transformación que deseas hacer a tu esquema, y el método `down` de tu migración debería revertir las transformaciones hechas por el método `up`. En otras palabras, el esquema de la base de datos permanecerá sin cambios si ejecutas un `up` seguido por un `down`. Por ejemplo, si creas una tabla en el método `up`, deberías borrarla en el método `down`. Esto es para revertir las transformaciones precisamente en el orden reverso en el que fueron hechas en el método `up`. El ejemplo en la sección reversible es equivalente a:

```ruby
class ExampleMigration < ActiveRecord::Migration
  def up
    create_table :distributors do |t|
      t.string :zipcode
    end
 
    # añadir una restricción CHECK
    execute <<-SQL
      ALTER TABLE distributors
        ADD CONSTRAINT zipchk
        CHECK (char_length(zipcode) = 5);
    SQL
 
    add_column :users, :home_page_url, :string
    rename_column :users, :email, :email_address
  end
  def down
    rename_column :users, :email_address, :email
    remove_column :users, :home_page_url
    execute <<-SQL
      ALTER TABLE distributors
        DROP CONSTRAINT zipchk
    SQL
 
    drop_table :distributors
  end
end
```

Si tu migración es irreversible, deberías lanzar una `ActiveRecord::IrreversibleMigration` desde tu método `down`. Si alguién trata de revertir la migración, un mensaje de error será mostrado diciendo que esto no se puede hacer.



### 3.11 Revertir Migraciones Anteriores

Puedes utilizar las posibilidades de Active Record para revertir migraciones utilizando el método revert:

```ruby
require_relative '2012121212_example_migration'
 
class FixupExampleMigration < ActiveRecord::Migration
  def change
    revert ExampleMigration
 
    create_table(:apples) do |t|
      t.string :variety
    end
  end
end
```

El método `revert` también acepta un bloque de instrucciones para revertir. Este puede ser utilizado para revertir partes seleccionadas de las migraciones. Por ejemplo, vamos a imaginar que la `ExampleMigration` es "commiteada" y más tarde se decide que lo mejor sería utilizar validaciones Active Record, en lugar de la restricción CHECK, para verificar el código postal \(zipcode\). 

```ruby
class DontUseConstraintForZipcodeValidationMigration < ActiveRecord::Migration
  def change
    revert do
      # código cortado y pedado de ExampleMigration
      reversible do |dir|
        dir.up do
        # add a CHECK constraint
          execute <<-SQL
            ALTER TABLE distributors
              ADD CONSTRAINT zipchk
                CHECK (char_length(zipcode) = 5);
          SQL
        end
        dir.down do
          execute <<-SQL
            ALTER TABLE distributors
              DROP CONSTRAINT zipchk
          SQL
        end
      end
 
      # El resto de la migración fue bien
    end
  end
end
```

La misma migración podría también haberse escrito sin utilizar `revert` pero esto podría haber llevado unos pocos pasos más: revertiendo el orden de `create_table` y `reversible`, reemplazando `create_table` por `drop_table`, y finalmente reemplazando `up` por `down` y vice-versa. Todo esto está a cargo de revert.

