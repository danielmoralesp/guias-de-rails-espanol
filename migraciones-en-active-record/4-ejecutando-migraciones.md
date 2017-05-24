# 4. Ejecutando Migraciones

Rails provee un conjunto de tareas Rake para ejecutar cierto conjunto de migraciones.

La primera tarea de migración que utilizarás será probablemente `rake db:migrate`. En su forma más básica ejecuta un método `change` o `up` para todas las migraciones que aún no se han ejecutado. Si no hay tales migraciones, finaliza. Ejecutará esas migraciones en orden basado en la fecha de creación de cada migración.

> Nota que ejecutar la tarea `db:migrate` también se invoca la tarea `db:schema:dump`, el cual actualizará tu fichero `db/schema.rb` para emparejarlo a la estructura de tu base de datos.

Si especificas una versión de destino, Active Record ejecturará las migraciones requeridas \(`change`, `up`, `down`\) hasta alcanzar la versión específica. La versión es el prefijo numérico en el nombre de fichero de la migración. Por ejemplo, para migrar a la versión 20080906120000 ejectuta:

```ruby
$ bin/rake db:migrate VERSION=20080906120000
```

Si la versión 20080906120000 es mayor que la versión actual, y se está migrando hacia arriba, se ejectutará el método `change` \(o `up`\) en todas las migraciones hacia arriba e incluirá la 20080906120000, y no ejecutará ninguna migración posterior. Si se está migrando hacia abajo, esto ejecutará los metodos `down` de todas las migraciones hacia abajo, paro no incluirá la 20080906120000.

### 4.1 Deshaciendo Migraciones

Una tarea común es deshacer la última migración. Por ejemplo, si cometes un error en esta y quieres corregirlo. En lugar de rastrear la versión anterior asociada puedes ejecutar:

```ruby
$ bin/rake db:rollback
```

Esto ejecutará hacia atrás la última migración, ya sea por revertir el método `change` o por ejecución del método `down`. Si necesitas deshacer varias migraciones puedes proveer un parámetro STEP:

```ruby
$ bin/rake db:rollback STEP=3
```

revertirá las últimas 3 migraciones.

La tarea `db:migrate:redo` es un atajo para deshacer y luego migrar hacia arriba otra vez. Con la tarea `db:rollback`, también puedes utilizar el parámetro STEP .

Si necesitas retroceder más de una versión haces lo siguiente, por ejemplo:

```ruby
$ bin/rake db:migrate:redo STEP=3
```

Ninguna de estas tareas Rake hace algo que no se pueda hacer con `db:migrate`. Son simplemente más convenientes, por que no necesitas escribir la versión específica desde la cual hay deshacer y volver a ejecutar las migraciones.

### 4.2 Configurar la Base de Datos

La tarea `rake db:setup` creará la base de datos, carga el esquema con los datos iniciales.

### 4.3 Recomponer la Base de Datos

La tarea `rake db:reset` borrará la base de datos y la configurará nuevamente. Esto es funcionalmente equivalente a `rake db:drop db:setup`.

Esto no es lo mismo que ejecutar todas las migraciones. Esto utilizará únicamente el fichero `schema.rb` actual. Si una migración no puede ser revertida, `rake db:reset` no te podrá ayudar. Para descubrir más acerca del volcado del esquema ver la sección El Volcado del Esquema y Tú.

### 4.4 Ejecutar Migraciones Específicas

Si necesitas ejecutar una migración específica hacia arriba o abajo, las tareas `db:migrate:up` y `db:migrate:down` harán esto. Sólo especifica la versión adecuada y la migración correspondiente tendrá su método `change`, `up` o `down` invocado, por ejemplo:

```ruby
$ bin/rake db:migrate:up VERSION=20080906120000
```

ejecutara la migración 20080906120000 por ejecución del método `change` \(o del método `up`\). Esta tarea primero comprobará si la migración está ya realizada y no hará nada si Active Record cree que esta ya se ha ejecutado antes.

### 4.5 Ejecutando Migraciones en Diferentes Entornos

Por defecto cuando ejecutamos `rake db:migrate` se ejecutará en el entorno de desarrollo o development. Si quieres ejecutar las migraciones otra vez en otro entorno, puedes especificarlo utilizando la variable de entorno `RAILS_ENV` cuando ejecutas el comando. Por ejemplo para ejecutar las migraciones nuevamente en el entorno de pruebas, test, podrías ejecutar:

```ruby
$ bin/rake db:migrate RAILS_ENV=test
```

### 4.6 Cambiando la Salida de la Ejecución de Migraciones

Por defecto las migraciones te indican exactamente que han hecho y cuanto tiempo le ha llevado. Una migración que crea una tabla y añade un índice puede producir una salida como esta:

```ruby
==  CreateProducts: migrating =================================================
-- create_table(:products)
   -> 0.0028s
==  CreateProducts: migrated (0.0028s) ========================================
```

Varios métodos son provistos en las migraciones que te permiten controlar todo esto:



| Metodo | Propósito |
| :--- | :--- |
| suppress\_messages | Toma un bloque como argumento y suprime cualquier salida generada por el bloque. |
| say | Toma como argumento un mensaje y emite tal cual es. Se le puede pasar un segundo argumento booleano para especificar si se va a tabular o no. |
| say\_with\_time | Salida de texto junto con el tiempo que tomó para ejecutar el bloque correspondiente. Si el bloque devuelve un entero que asume que es el número de filas afectadas. |



Por ejemplo, esta migración:

```ruby
class CreateProducts < ActiveRecord::Migration
  def change
    suppress_messages do
      create_table :products do |t|
        t.string :name
        t.text :description
        t.timestamps null: false
      end
    end
 
    say "Created a table"
    suppress_messages {add_index :products, :name}
    say "and an index!", true
 
    say_with_time 'Waiting for a while' do
      sleep 10
      250
    end
  end
end
```

genera la siguiente salida

```ruby
==  CreateProducts: migrating =================================================
-- Created a table
   -> and an index!
-- Waiting for a while
   -> 10.0013s
   -> 250 rows
==  CreateProducts: migrated (10.0054s) =======================================
```

Si quieres que Active Record no muestre ninguna salida, ejecutando `rake db:migrate VERBOSE=false` se suprimirán toda las salidas.



