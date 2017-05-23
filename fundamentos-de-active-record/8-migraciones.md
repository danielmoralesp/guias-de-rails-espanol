# 8. Migraciones

Rails provee un lenguaje de dominio específico para manejar un esquema de base de datos llamado migraciones \(migrations\). Las migraciones son ficheros guardados que se ejecutan contra cualquier base de datos que Active Record soporte utilizando rake. Aquí hay una migración que crea una tabla:

```ruby
class CreatePublications < ActiveRecord::Migration
  def change
    create_table :publications do |t|
      t.string :title
      t.text :description
      t.references :publication_type
      t.integer :publisher_id
      t.string :publisher_type
      t.boolean :single_issue
 
      t.timestamps null: false
    end
    add_index :publications, :publication_type_id
  end
end
```

Rails mantiene el historial sobre que fichero fue actualizado en la base de datos y provee características para deshacer los cambios. Para realmente crear la tabla, deberías ejecutar `rake db:migrate` y para deshacerlo `rake db:rollback`. 

> Nota que el código de arriba es database-agnostic: se puede ejectuar en MySQL, PostgreSQL, Oracle y others. Puedes aprender más acerca de las migraciones en la Guía de Migraciones Active Record.



