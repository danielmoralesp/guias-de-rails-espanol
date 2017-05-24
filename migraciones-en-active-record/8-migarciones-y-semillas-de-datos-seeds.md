# 8. Migraciones y Semillas de Datos \(Seeds\)

Algunas personas utilizan las migraciones para añadir datos a la base de datos:

```ruby
class AddInitialProducts < ActiveRecord::Migration
  def up
    5.times do |i|
      Product.create(name: "Product ##{i}", description: "A product.")
    end
  end
 
  def down
    Product.delete_all
  end
end
```

Sin embargo, Rails tiene una caraterística 'seeds' que puede ser utilizada para crear las semillas en una base de datos con datos iniciales. Esta es una característica realmente simple: solo rellena `db/seeds.rb` con algún codigo Ruby, y ejecuta `rake db:seed:`

```ruby
5.times do |i|
  Product.create(name: "Product ##{i}", description: "A product.")
end
```

Esto es generalmente el camino más limpio para configurar la base de datos de una aplicación en blanco.

