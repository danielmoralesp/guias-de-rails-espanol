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















