# 1- Introducción

Active Model es una biblioteca que contiene varios módulos utilizados en el desarrollo de clases que necesitan algunas características presentes en Active Record. A continuación se explican algunos de estos módulos.



### 1.1 Métodos de atributo

El módulo `ActiveModel::AttributeMethods` puede agregar prefijos y sufijos personalizados en los métodos de una clase. Se utiliza mediante la definición de los prefijos y sufijos y qué métodos en el objeto se utilizan.

```ruby
class Person
  include ActiveModel::AttributeMethods
 
  attribute_method_prefix 'reset_'
  attribute_method_suffix '_highest?'
  define_attribute_methods 'age'
 
  attr_accessor :age
 
  private
    def reset_attribute(attribute)
      send("#{attribute}=", 0)
    end
 
    def attribute_highest?(attribute)
      send(attribute) > 100
    end
end
 
person = Person.new
person.age = 110
person.age_highest?  # => true
person.reset_age     # => 0
person.age_highest?  # => false
```



### 1.2 Devoluciones de llamada

`ActiveModel::Callbacks` ofrece devoluciones de llamada al estilo de Active Record. Esto proporciona una capacidad para definir devoluciones de llamada que se ejecutan en momentos apropiados. Después de definir las devoluciones de llamada, puede envolverlas con métodos personalizados antes, durante y después.

```ruby
class Person
  extend ActiveModel::Callbacks
 
  define_model_callbacks :update
 
  before_update :reset_me
 
  def update
    run_callbacks(:update) do
      # This method is called when update is called on an object.
    end
  end
 
  def reset_me
    # This method is called when update is called on an object as a before_update callback is defined.
  end
end
```



### 1.3 Conversion

Si una clase define `persisted?` y el método `id`, puede incluir el módulo `ActiveModel::Conversion` en esa clase y llamar a los métodos de conversión de Rails en objetos de esa clase.

```ruby
class Person
  include ActiveModel::Conversion
 
  def persisted?
    false
  end
 
  def id
    nil
  end
end
 
person = Person.new
person.to_model == person  # => true
person.to_key              # => nil
person.to_param            # => nil
```



### 1.4 Dirty

Un objeto se ensucia cuando ha pasado por uno o más cambios en sus atributos y no se ha guardado. `ActiveModel::Dirty `permite comprobar si un objeto ha sido cambiado o no. También tiene métodos de acceso basados en atributo. Consideremos una clase `Person` con los atributos `first_name` y `last_name`:

```ruby
class Person
  include ActiveModel::Dirty
  define_attribute_methods :first_name, :last_name
 
  def first_name
    @first_name
  end
 
  def first_name=(value)
    first_name_will_change!
    @first_name = value
  end
 
  def last_name
    @last_name
  end
 
  def last_name=(value)
    last_name_will_change!
    @last_name = value
  end
 
  def save
    # do save work...
    changes_applied
  end
end
```













