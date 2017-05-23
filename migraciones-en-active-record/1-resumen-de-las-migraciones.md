# 1. Resumen de las Migraciones

Las migraciones son el modo conveniente de cambiar el esquema de tu base de datos a través del tiempo de una manera consistente y fácil. Ellas utilizan un lenguaje de Definición de Esquemas \(DSL\) en Ruby por lo que no tienes que escribir SQL a mano, permitiéndole al esquema y a los cambios en la base de datos ser independientes. Puedes pensar cada migración como una nueva 'versión' de la base de datos. Un esquema comienza sin nada dentro, y cada migración lo modifica para añadir o remover tablas, columnas, o registros. Active Record conoce como actualizar tu esquema a lo largo de su vida, trayendo desde cualquier punto de su historia hasta la última versión. Active Record actualizará también el fichero `db/schema.rb` para emparejar la estructura modificada de tu base de datos.

Aquí un ejemplo de migración:

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

Esta migración añade una tabla llamada `products` con una columna de cadena de caractéres llamada `name` y una columna de texto llamada `description`. Una columna de clave primaria llamada `id` será también añadida implícitamente, como la clave primaria por defecto para todos los modelos Active Record. Los macro `timestamps` añaden dos columnas, `created_at` y `updated_at`. Estas columnas especiales son automaticamemente administradas por Active Record si existen.

Nota que nosotros definimos el cambio que queremos que ocurra a través del tiempo. Antes de que esta migración se ejecute, no existía la tabla, después de la migración la tabla existirá. Active Record, tambien sabe como revertir esta migración. Si ejecutamos la migración hacia atrás, borrará la tabla. En las bases de datos que soportan transacciones con declaraciones que cambian el esquema, las migraciones son envueltas en una transacción. Si la base de datos no soporta esto, cuando una migración falla la parte de esta que ha tenido éxito no se vuelve atrás. Tendrás que hacer el retroceso de los cambios que fueron hechos a mano.

Hay ciertas consultas que no pueden ejecutarse en una transacción. Si tu adaptador soporta transacciones DSL puedes utilizar `disable_ddl_transaction!` para deshabilitarlas para una simple migración.

Si tu deseas en una migración hacer algo que Active Record no sabe como revertir, puedes utilizar `reversible`:

```ruby
class ChangeProductsPrice < ActiveRecord::Migration
  def change
    reversible do |dir|
      change_table :products do |t|
        dir.up   { t.change :price, :string }
        dir.down { t.change :price, :integer }
      end
    end
  end
end
```

Alternativamente, puedes utilizar `up` y `down` en lugar de `change`:

```ruby
class ChangeProductsPrice < ActiveRecord::Migration
  def up
    change_table :products do |t|
      t.change :price, :string
    end
  end

  def down
    change_table :products do |t|
      t.change :price, :integer
    end
  end
end
```



