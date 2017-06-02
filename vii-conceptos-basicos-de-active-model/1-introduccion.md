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

Un objeto se ensucia cuando ha pasado por uno o más cambios en sus atributos y no se ha guardado. `ActiveModel::Dirty`permite comprobar si un objeto ha sido cambiado o no. También tiene métodos de acceso basados en atributo. Consideremos una clase `Person` con los atributos `first_name` y `last_name`:

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

#### 1.4.1 Consultando el objeto directamente para su lista de todos los atributos cambiados.

```ruby
person = Person.new
person.changed? # => false
 
person.first_name = "First Name"
person.first_name # => "First Name"
 
# returns true if any of the attributes have unsaved changes.
person.changed? # => true
 
# returns a list of attributes that have changed before saving.
person.changed # => ["first_name"]
 
# returns a Hash of the attributes that have changed with their original values.
person.changed_attributes # => {"first_name"=>nil}
 
# returns a Hash of changes, with the attribute names as the keys, and the
# values as an array of the old and new values for that field.
person.changes # => {"first_name"=>[nil, "First Name"]}
```

#### 1.4.2 Métodos de acceso basado en atributos

```ruby
# attr_name_changed?
person.first_name # => "First Name"
person.first_name_changed? # => true
```

Rastrea el valor anterior del atributo.

```ruby
# attr_name_was accessor
person.first_name_was # => nil
```

Rastrea el valor anterior y el actual del atributo cambiado. Devuelve una matriz si se cambia, de lo contrario devuelve nil.

```ruby
# attr_name_change
person.first_name_change # => [nil, "First Name"]
person.last_name_change # => nil
```



### 1.5 Validaciones

El módulo `ActiveModel::Validations` agrega la capacidad de validar objetos como en Active Record.

```ruby
class Person
  include ActiveModel::Validations
 
  attr_accessor :name, :email, :token
 
  validates :name, presence: true
  validates_format_of :email, with: /\A([^\s]+)((?:[-a-z0-9]\.)[a-z]{2,})\z/i
  validates! :token, presence: true
end
 
person = Person.new
person.token = "2b1f325"
person.valid?                        # => false
person.name = 'vishnu'
person.email = 'me'
person.valid?                        # => false
person.email = 'me@vishnuatrai.com'
person.valid?                        # => true
person.token = nil
person.valid?                        # => raises ActiveModel::StrictValidationFailed
```



### 1.6 Nombrar

`ActiveModel::Naming` añade una serie de métodos de clase que facilitan la gestión de nomenclatura y enrutamiento. El módulo define el método de clase `model_name` que definirá un número de `accessors` usando algunos métodos `ActiveSupport::Inflector`.

```ruby
class Person
  extend ActiveModel::Naming
end
 
Person.model_name.name                # => "Person"
Person.model_name.singular            # => "person"
Person.model_name.plural              # => "people"
Person.model_name.element             # => "person"
Person.model_name.human               # => "Person"
Person.model_name.collection          # => "people"
Person.model_name.param_key           # => "person"
Person.model_name.i18n_key            # => :person
Person.model_name.route_key           # => "people"
Person.model_name.singular_route_key  # => "person"
```



### 1.7 Modelo

`ActiveModel::Model` añade la capacidad de una clase para trabajar con Action Pack y Action View justo fuera de la caja.

```ruby
class EmailContact
  include ActiveModel::Model
 
  attr_accessor :name, :email, :message
  validates :name, :email, :message, presence: true
 
  def deliver
    if valid?
      # deliver email
    end
  end
end
```

Al incluir `ActiveModel::Model` obtienes algunas características como:

* model name introspection
* conversions
* translation
* validations

También le da la capacidad de inicializar un objeto con un hash de atributos, al igual que cualquier objeto de Active Record.

```ruby
email_contact = EmailContact.new(name: 'David',
                                 email: 'david@example.com',
                                 message: 'Hello World')
email_contact.name       # => 'David'
email_contact.email      # => 'david@example.com'
email_contact.valid?     # => true
email_contact.persisted? # => false
```

Cualquier clase que incluya `ActiveModel::Model` se puede utilizar con `form_for`, `render` y cualquier otro método de ayuda Action View, al igual que los objetos Active Record.



### 1.8 Serialización

`ActiveModel::Serialization` proporciona serialización básica para su objeto. Debe declarar un Hash de atributos que contiene los atributos que desea serializar. Los atributos deben ser cadenas, no símbolos.

```ruby
class Person
  include ActiveModel::Serialization
 
  attr_accessor :name
 
  def attributes
    {'name' => nil}
  end
end
```

Ahora puede acceder a un Hash serializado de su objeto utilizando el método `serializable_hash`.

```ruby
person = Person.new
person.serializable_hash   # => {"name"=>nil}
person.name = "Bob"
person.serializable_hash   # => {"name"=>"Bob"}
```

#### 1.8.1 ActiveModel::Serializers

Active Model también proporciona el módulo `ActiveModel::Serializers::JSON` para la serialización / deserialización de JSON. Este módulo incluye automáticamente el módulo de `ActiveModel::Serialization` anteriormente discutido.

##### 1.8.1.1 ActiveModel::Serializers::JSON

Para utilizar `ActiveModel::Serializers::JSON` sólo necesita cambiar el módulo que está incluyendo de `ActiveModel::Serialization` a `ActiveModel::Serializers::JSON`.

```ruby
class Person
  include ActiveModel::Serializers::JSON
 
  attr_accessor :name
 
  def attributes
    {'name' => nil}
  end
end
```

El método `as_json`, similar a `serializable_hash`, proporciona un Hash que representa el modelo.

```ruby
person = Person.new
person.as_json # => {"name"=>nil}
person.name = "Bob"
person.as_json # => {"name"=>"Bob"}
```

También puede definir los atributos para un modelo de una cadena JSON. Sin embargo, debe definir el método `attributes=` en su clase:

```ruby
class Person
  include ActiveModel::Serializers::JSON
 
  attr_accessor :name
 
  def attributes=(hash)
    hash.each do |key, value|
      send("#{key}=", value)
    end
  end
 
  def attributes
    {'name' => nil}
  end
end
```

Ahora es posible crear una instancia de `Person` y establecer atributos usando `from_json`.

```ruby
json = { name: 'Bob' }.to_json
person = Person.new
person.from_json(json) # => #<Person:0x00000100c773f0 @name="Bob">
person.name            # => "Bob"
```



### 1.9 Traducción

`ActiveModel::Translation` proporciona integración entre su objeto y el entorno de internacionalización de Rails \(i18n\).

```ruby
class Person
  extend ActiveModel::Translation
end
```

Con el método `human_attribute_name`, puede transformar los nombres de los atributos en un formato más legible por el ser humano. El formato legible por el usuario se define en su \(s\) archivo \(s\) local \(es\).

* config/locales/app.pt-BR.yml

```ruby
pt-BR:
  activemodel:
    attributes:
      person:
        name: 'Nome'
```

```ruby
Person.human_attribute_name('name') # => "Nome"
```



### 1.10 Lint Tests

`ActiveModel::Lint::Tests` le permite comprobar si un objeto cumple con la API del Active Model.

* app/models/person.rb

```ruby
class Person
  include ActiveModel::Model
end
```

* test/models/person\_test.rb

```ruby
require 'test_helper'
 
class PersonTest < ActiveSupport::TestCase
  include ActiveModel::Lint::Tests
 
  setup do
    @model = Person.new
  end
end
```

```ruby
$ rails test
 
Run options: --seed 14596
 
# Running:
 
......
 
Finished in 0.024899s, 240.9735 runs/s, 1204.8677 assertions/s.
 
6 runs, 30 assertions, 0 failures, 0 errors, 0 skips
```

No se requiere un objeto para implementar todas las API para trabajar con Action Pack. Este módulo sólo tiene la intención de proporcionar orientación en caso de que desee todas las características fuera de la caja.



### 1.11 SecurePassword

`ActiveModel::SecurePassword` proporciona una forma de almacenar de forma segura cualquier contraseña en un formato cifrado. Cuando se incluye este módulo, se proporciona un método de clase `has_secure_password` que define un `accesor` de contraseña con ciertas validaciones en él.

#### 1.11.1 Requisitos

`ActiveModel::SecurePassword` depende de `bcrypt`, así que incluya esta gema en su `Gemfile` para usar `ActiveModel::SecurePassword` correctamente. Para que esto funcione, el modelo debe tener un `accesor` llamado `password_digest`. El `has_secure_password` agregará las siguientes validaciones en el `accesor` de contraseña:

1. La contraseña debe estar presente.
2. La contraseña debe ser igual a su confirmación \(se proporciona + password\_confirmation\).
3. La longitud máxima de una contraseña es 72 \(requerida por `bcrypt` de la que depende `ActiveModel::SecurePassword`\)

##### 1.11.2 Ejemplos

```ruby
class Person
  include ActiveModel::SecurePassword
  has_secure_password
  attr_accessor :password_digest
end
 
person = Person.new
 
# When password is blank.
person.valid? # => false
 
# When the confirmation doesn't match the password.
person.password = 'aditya'
person.password_confirmation = 'nomatch'
person.valid? # => false
 
# When the length of password exceeds 72.
person.password = person.password_confirmation = 'a' * 100
person.valid? # => false
 
# When only password is supplied with no password_confirmation.
person.password = 'aditya'
person.valid? # => true
 
# When all validations are passed.
person.password = person.password_confirmation = 'aditya'
person.valid? # => true
```



