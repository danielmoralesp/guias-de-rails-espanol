# 4. Sobre-escribiendo las Convenciones de Nombres

¿Que sucede si quieres seguir diferentes convenciones de nombres o necesitas utilizar una aplicación Rails con una base de datos heredada? No hay problema, puedes fácilmente sobre-escribir las convenciones por defecto. Utilizando el método `ActiveRecord::Base.table_name=` para especificar el nombre de la tabla que debería ser utilizada:

```ruby
class Product < ApplicationRecord
    self.table_name = "my_products"
end
```

Si tu haces esto, tendrás que definir manualmente el nombre de la clase a contiene los fixtures \(`class_name.yml`\) utilizando el método the `set_fixture_class` en la definición de los test.

```ruby
class FunnyJoke < ActiveSupport::TestCase
  set_fixture_class funny_jokes: Joke
  fixtures :funny_jokes
  ...
end
```

También es posible cambiar la columna clave primaria de la tabla con el método `ActiveRecord::Base.primary_key=`:

```ruby
class Product < ActiveRecord::Base
  self.primary_key = "product_id"
end
```



