# 1. ¿Por qué asociaciones?

En Rails, una asociación es una conexión entre dos modelos Active Record. ¿Por qué necesitamos las asociaciones entre modelos? Porque hacen las operaciones comunes más simples y más fáciles en su código. Por ejemplo, considere una sencilla aplicación de Rails que incluye un modelo para autores y un modelo para libros. Cada autor puede tener muchos libros. Sin asociaciones, las declaraciones del modelo se verían así:

```ruby
class Author < ApplicationRecord
end
 
class Book < ApplicationRecord
end
```

Ahora, supongamos que queremos agregar un nuevo libro para un autor existente. Necesitamos hacer algo como esto:

```ruby
@book = Book.create(published_at: Time.now, author_id: @author.id)
```

O considere eliminar un autor y asegurarse de que todos sus libros también se eliminen:

```ruby
@books = Book.where(author_id: @author.id)
@books.each do |book|
  book.destroy
end
@author.destroy
```

Con las asociaciones Active Record, podemos agilizar estas y otras operaciones declarando a Rails que hay una conexión entre los dos modelos. Aquí está el código revisado para la creación de autores y libros:

```ruby
class Author < ApplicationRecord
  has_many :books, dependent: :destroy
end
 
class Book < ApplicationRecord
  belongs_to :author
end
```

Con este cambio, crear un nuevo libro para un autor en particular es más fácil:

```ruby
@book = @author.books.create(published_at: Time.now)
```

Eliminar un autor y todos sus libros es mucho más fácil:

```ruby
@author.destroy
```

Para obtener más información sobre los diferentes tipos de asociaciones, lea la siguiente sección de esta guía. A continuación, algunos consejos y trucos para trabajar con asociaciones y, a continuación, una referencia completa a los métodos y opciones para las asociaciones en Rails.

