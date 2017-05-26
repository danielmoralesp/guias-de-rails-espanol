# 4.6 Extensiones de la asociación

No se limita a la funcionalidad que Rails crea automáticamente en objetos proxy de asociación. También puede ampliar estos objetos a través de módulos anónimos, agregando nuevos buscadores, creadores u otros métodos. Por ejemplo:

```ruby
class Author < ApplicationRecord
  has_many :books do
    def find_by_book_prefix(book_number)
      find_by(category_id: book_number[0..2])
    end
  end
end
```

Si tiene una extensión que debe ser compartida por muchas asociaciones, puede utilizar un módulo de extensión con nombre. Por ejemplo:

```ruby
module FindRecentExtension
  def find_recent
    where("created_at > ?", 5.days.ago)
  end
end
 
class Author < ApplicationRecord
  has_many :books, -> { extending FindRecentExtension }
end
 
class Supplier < ApplicationRecord
  has_many :deliveries, -> { extending FindRecentExtension }
end
```

Las extensiones pueden referirse a los internos del proxy de asociación utilizando estos tres atributos del `accesor` de `proxy_association`:

* `proxy_association.owner` devuelve el objeto al que pertenece la asociación. 
* `proxy_association.reflection` devuelve el objeto de reflexión que describe la asociación. 
* `proxy_association.target` devuelve el objeto asociado para `belongs_to` o `has_one`, o la colección de objetos asociados para `has_many` o `has_and_belongs_to_many`.





