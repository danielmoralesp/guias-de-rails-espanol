# 16. Enums

La macro de enumeración asigna una columna de número entero a un conjunto de valores posibles.

```ruby
class Book < ApplicationRecord
  enum availability: [:available, :unavailable]
end
```

Esto creará automáticamente los ámbitos correspondientes para consultar el modelo. Los métodos de transición entre los estados y la consulta del estado actual también se agregan.

```ruby
# Ambos ejemplos a continuación consultan sólo los libros disponibles.
Book.available
# or
Book.where(availability: :available)
 
book = Book.new(availability: :available)
book.available?   # => true
book.unavailable! # => true
book.available?   # => false
```

Lea la documentación completa sobre enums en los documentos [de la API de Rails](http://api.rubyonrails.org/v5.1.1/classes/ActiveRecord/Enum.html).



