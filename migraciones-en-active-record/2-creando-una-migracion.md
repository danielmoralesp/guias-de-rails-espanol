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



