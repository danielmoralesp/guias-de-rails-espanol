# 4.5 Callbacks de la asociación

Las callbacks normales se conectan al ciclo de vida de los objetos Active Record, permitiéndole trabajar con esos objetos en varios puntos. Por ejemplo, puede utilizar una callbacks `before_save` para hacer que algo suceda justo antes de guardar un objeto. 

Las callbacks de la asociación son similares a las callbacks normales, pero son activadas por eventos en el ciclo de vida de una colección. Existen cuatro callbacks de asociación disponibles:

```ruby
before_add
after_add
before_remove
after_remove
```

Define las callbacks de asociación agregando opciones a la declaración de asociación. Por ejemplo:

```ruby
class Author < ApplicationRecord
  has_many :books, before_add: :check_credit_limit
 
  def check_credit_limit(book)
    ...
  end
end
```

Rails pasa el objeto que se agrega o se elimina a la callback.

Puede apilar callbacks en un solo evento pasándolas como una matriz:

```ruby
class Author < ApplicationRecord
  has_many :books,
    before_add: [:check_credit_limit, :calculate_shipping_charges]
 
  def check_credit_limit(book)
    ...
  end
  def calculate_shipping_charges(book)
    ...
  end
end
```

Si una callback `before_add` lanza una excepción, el objeto no se agrega a la colección. Del mismo modo, si una callback `before_remove` produce una excepción, el objeto no se quita de la colección.

