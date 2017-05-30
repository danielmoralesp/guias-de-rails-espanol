# 14. Ambitos

Scoping le permite especificar consultas de uso común que pueden ser referenciadas como llamadas de método en los objetos o modelos de asociación. Con estos ámbitos, puede utilizar todos los métodos cubiertos anteriormente, como `where`, `join` e `includes`. Todos los métodos de ámbito devolverán un objeto `ActiveRecord::Relation` que permitirá que se apliquen otros métodos \(o otros ámbitos\).

Para definir un ámbito simple, utilizamos el método `scope` dentro de la clase, pasando la consulta que queremos ejecutar cuando se llama a este ámbito:

```ruby
class Article < ApplicationRecord
  scope :published, -> { where(published: true) }
end
```

Esto es exactamente lo mismo que definir un método de clase, y cuál se utiliza es una cuestión de preferencia personal:

```ruby
class Article < ApplicationRecord
  def self.published
    where(published: true)
  end
end
```

Los ámbitos también se pueden encadenar dentro de otros ámbitos:

```ruby
class Article < ApplicationRecord
  scope :published,               -> { where(published: true) }
  scope :published_and_commented, -> { published.where("comments_count > 0") }
end
```

Para llamar al ámbito `published`, podemos llamarlo en la clase:

```ruby
Article.published # => [published articles]
```

O en una asociación que consiste en objetos del `Article`:

```ruby
category = Category.first
category.articles.published # => [Artículos publicados pertenecientes a esta categoría]
```



### 14.1 Pasando argumentos

Su `scope` puede tomar argumentos:

```ruby
class Article < ApplicationRecord
  scope :created_before, ->(time) { where("created_at < ?", time) }
end
```

Llama al ámbito como si fuera un método de clase:

```ruby
Article.created_before(Time.zone.now)
```

Sin embargo, esto es sólo la duplicación de la funcionalidad que se le proporcionaría a usted por medio de un método de clase.

```ruby
class Article < ApplicationRecord
  def self.created_before(time)
    where("created_at < ?", time)
  end
end
```

El uso de un método de clase es la forma preferida de aceptar argumentos para ámbitos. Estos métodos seguirán siendo accesibles en los objetos de la asociación:

```ruby
category.articles.created_before(time)
```



### 14.2 Uso de condicionales

Su `scope` puede utilizar condicionales:

```ruby
class Article < ApplicationRecord
  scope :created_before, ->(time) { where("created_at < ?", time) if time.present? }
end
```

Al igual que los otros ejemplos, esto se comportará de manera similar a un método de clase.

```ruby
class Article < ApplicationRecord
  def self.created_before(time)
    where("created_at < ?", time) if time.present?
  end
end
```

Sin embargo, hay una advertencia importante: Un `scope` siempre devolverá un objeto `ActiveRecord::Relation`, incluso si el condicional se evalúa como `false`, mientras que un método de clase devolverá `nil`. Esto puede causar `NoMethodError` al encadenar métodos de clase con condicionales, si cualquiera de los condicionales devuelve `false`.



### 14.3 Aplicación de un ámbito predeterminado

Si deseamos que se aplique un `scope` a todas las consultas del modelo, podemos utilizar el método `default_scope` dentro del propio modelo.

```ruby
class Client < ApplicationRecord
  default_scope { where("removed_at IS NULL") }
end
```

Cuando las consultas se ejecutan en este modelo, la consulta `SQL` ahora se verá como esto:

```ruby
SELECT * FROM clients WHERE removed_at IS NULL
```

Si necesita hacer cosas más complejas con un ámbito predeterminado, puede definirlo como un método de clase:

```ruby
class Client < ApplicationRecord
  def self.default_scope
    # Debe devolver un ActiveRecord::Relation.
  end
end
```

> El default\_scope también se aplica durante la creación/construcción de un registro. No se aplica durante la actualización de un registro. P.ej.:



```ruby
class Client < ApplicationRecord
  default_scope { where(active: true) }
end
 
Client.new          # => #<Client id: nil, active: true>
Client.unscoped.new # => #<Client id: nil, active: nil>
```



### 14.4 Fusión de los ámbitos

Funciona igual que cuando usamos las cláusulas de los ámbitos que se combinan utilizando las condiciones `AND`.

```ruby
class User < ApplicationRecord
  scope :active, -> { where state: 'active' }
  scope :inactive, -> { where state: 'inactive' }
end
 
User.active.inactive
# SELECT "users".* FROM "users" WHERE "users"."state" = 'active' AND "users"."state" = 'inactive'
```











