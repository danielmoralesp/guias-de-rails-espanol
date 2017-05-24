# 7. Trabajar con errores de validación

Además de los Métodos `valid?` e `invalid?` cubiertos anteriormente, Rails proporciona una serie de métodos para trabajar con la colección de errores y preguntar sobre la validez de los objetos. 

La siguiente es una lista de los métodos más utilizados. Consulte la documentación de `ActiveModel::Errors` para obtener una lista de todos los métodos disponibles. 



### 7.1 errors 

Devuelve una instancia de la clase `ActiveModel::Errors` que contiene todos los errores. Cada clave es el nombre del atributo y el valor es una matriz de cadenas con todos los errores.

```ruby
class Person < ActiveRecord::Base
  validates :name, presence: true, length: { minimum: 3 }
end
 
person = Person.new
person.valid? # => false
person.errors.messages
 # => {:name=>["can't be blank", "is too short (minimum is 3 characters)"]}
 
person = Person.new(name: "John Doe")
person.valid? # => true
person.errors.messages # => {}
```



### 7.2 errors\[ \] 

`errors[ ]` se utiliza cuando se desea comprobar los mensajes de error de un atributo específico. Devuelve una matriz de cadenas con todos los mensajes de error para el atributo dado, cada cadena con un mensaje de error. Si no hay errores relacionados con el atributo, devuelve una matriz vacía.

```ruby
class Person < ActiveRecord::Base
  validates :name, presence: true, length: { minimum: 3 }
end
 
person = Person.new(name: "John Doe")
person.valid? # => true
person.errors[:name] # => []
 
person = Person.new(name: "JD")
person.valid? # => false
person.errors[:name] # => ["is too short (minimum is 3 characters)"]
 
person = Person.new
person.valid? # => false
person.errors[:name]
 # => ["can't be blank", "is too short (minimum is 3 characters)"]
```



### 7.3 errors.add 

El método `add` le permite agregar manualmente mensajes relacionados con atributos particulares. Puede utilizar los métodos `errors.full_messages` o `errors.to_a` para ver los mensajes en la forma en que se pueden mostrar a un usuario. Esos mensajes particulares obtienen el nombre de atributo prependido \(y capitalizado\). `add` recibe el nombre del atributo al que desea agregar el mensaje y el propio mensaje.

```ruby
class Person < ActiveRecord::Base
  def a_method_used_for_validation_purposes
    errors.add(:name, "cannot contain the characters !@#%*()_-+=")
  end
end
 
person = Person.create(name: "!@#")
 
person.errors[:name]
 # => ["cannot contain the characters !@#%*()_-+="]
 
person.errors.full_messages
 # => ["Name cannot contain the characters !@#%*()_-+="]
```

Otra forma de hacerlo es utilizando `[] = setter` 

```ruby
class Person < ActiveRecord::Base
  def a_method_used_for_validation_purposes
    errors[:name] = "cannot contain the characters !@#%*()_-+="
  end
end
 
person = Person.create(name: "!@#")
 
person.errors[:name]
 # => ["cannot contain the characters !@#%*()_-+="]
 
person.errors.to_a
 # => ["Name cannot contain the characters !@#%*()_-+="]
```



### 7.4 errors\[: base\] 

Puede agregar mensajes de error relacionados con el estado del objeto como un todo, en lugar de estar relacionado con un atributo específico. Puede utilizar este método cuando desea decir que el objeto no es válido, independientemente de los valores de sus atributos. Como `errors[: base]` es un array, simplemente puede agregar una cadena a ella y se utilizará como un mensaje de error.

```ruby
class Person < ActiveRecord::Base
  def a_method_used_for_validation_purposes
    errors[:base] << "This person is invalid because ..."
  end
end
```



### 7.5 errors.clear 

El método `clear` se utiliza cuando intencionalmente desea borrar todos los mensajes de la colección de errores. Por supuesto, llamar a `errors.clear` sobre un objeto no válido en realidad no lo hará `valid:` la colección de errores ahora estará vacía, pero la próxima vez que llame `valid?` O cualquier método que intente guardar este objeto en la base de datos, las validaciones se ejecutarán de nuevo. Si alguna de las validaciones falla, la colección de errores se rellenará de nuevo.

```ruby
class Person < ActiveRecord::Base
  validates :name, presence: true, length: { minimum: 3 }
end

person = Person.new
person.valid? # => false
person.errors[:name]
 # => ["can't be blank", "is too short (minimum is 3 characters)"]
 
person.errors.clear
person.errors.empty? # => true
 
p.save # => false
 
p.errors[:name]
# => ["can't be blank", "is too short (minimum is 3 characters)"]
```



### 7.6 errors.size 

El método `size` devuelve el número total de mensajes de error para el objeto.

```ruby
class Person < ActiveRecord::Base
  validates :name, presence: true, length: { minimum: 3 }
end

person = Person.new
person.valid? # => false
person.errors.size # => 2
 
person = Person.new(name: "Andrea", email: "andrea@example.com")
person.valid? # => true
person.errors.size # => 0
```













