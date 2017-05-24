# 3. Opciones Comunes de Validación

Estas son las opciones comunes de validación: 

### 3.1 :allow\_nil

La opción: `allow_nil` omite la validación cuando el valor que se valida es `nil`.

```ruby
class Coffee < ActiveRecord::Base
  validates :size, inclusion: { in: %w(small medium large),
    message: "%{value} is not a valid size" }, allow_nil: true
end
```



### 3.2 :allow\_blank 

La opción: `allow_blank` es similar a la opción: `allow_nil`. Esta opción permitirá que la validación pase si el valor del atributo está en `blank?`, como `nil` o una cadena vacía por ejemplo.

```ruby
class Topic < ActiveRecord::Base
  validates :title, length: { is: 5 }, allow_blank: true
end
 
Topic.create(title: "").valid?  # => true
Topic.create(title: nil).valid? # => true
```



### 3.3 :message 

Como ya lo ha visto, la opción: `message` le permite especificar el mensaje que se agregará a la colección de errores cuando la validación falle. Cuando no se utiliza esta opción, Active Record utilizará el mensaje de error predeterminado respectivo para cada ayudante de validación.



### 3.4 :on 

La opción `:on` permite especificar cuándo debe ocurrir la validación. El comportamiento predeterminado de todos los ayudantes de validación incorporados se ejecutará en la función Guardar \(tanto cuando se crea un nuevo registro como cuando se está actualizando\). Si desea cambiarlo, puede utilizar `on::create` para ejecutar la validación sólo cuando se crea un nuevo registro o `on::update` para ejecutar la validación sólo cuando se actualice un registro.

```ruby
class Person < ActiveRecord::Base
  # it will be possible to update email with a duplicated value
  validates :email, uniqueness: true, on: :create
 
  # it will be possible to create the record with a non-numerical age
  validates :age, numericality: true, on: :update
 
  # the default (validates on both create and update)
  validates :name, presence: true
end
```



