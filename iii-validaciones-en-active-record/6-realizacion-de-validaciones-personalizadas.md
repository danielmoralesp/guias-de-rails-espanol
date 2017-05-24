# 6. Realización de validaciones personalizadas

Cuando los ayudantes de validación incorporados no son suficientes para sus necesidades, puede escribir sus propios validadores o métodos de validación como prefiera.



### 6.1 Validadores personalizados 

Los validadores personalizados son clases que amplían `ActiveModel::Validator`. Estas clases deben implementar un método de validación que toma un registro como un argumento y realiza la validación en él. El validador personalizado se llama utilizando el método `validates_with`.

```ruby
class MyValidator < ActiveModel::Validator
  def validate(record)
    unless record.name.starts_with? 'X'
      record.errors[:name] << 'Need a name starting with X please!'
    end
  end
end
 
class Person
  include ActiveModel::Validations
  validates_with MyValidator
end
```

La manera más fácil de agregar validadores personalizados para validar atributos individuales es con el conveniente `ActiveModel::EachValidator`. En este caso, la clase `validator` personalizada debe implementar un método `validate_each` que toma tres argumentos: `record`, `attribute` y `value`. Éstos corresponden a la instancia, el atributo a validar y el valor del atributo en la instancia pasada.

```ruby
class EmailValidator < ActiveModel::EachValidator
  def validate_each(record, attribute, value)
    unless value =~ /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i
      record.errors[attribute] << (options[:message] || "is not an email")
    end
  end
end
 
class Person < ActiveRecord::Base
  validates :email, presence: true, email: true
end
```

Como se muestra en el ejemplo, también puede combinar validaciones estándar con sus propios validadores personalizados.



### 6.2 Métodos personalizados 

También puede crear métodos que verifiquen el estado de sus modelos y agreguen mensajes a la colección de errores cuando no sean válidos. A continuación, debe registrar estos métodos utilizando el método `validate class`, pasando los símbolos para los nombres de los métodos de validación. Puede pasar más de un símbolo para cada método de clase y las respectivas validaciones se ejecutarán en el mismo orden en que se registraron.

```ruby
class Invoice < ActiveRecord::Base
  validate :expiration_date_cannot_be_in_the_past,
    :discount_cannot_be_greater_than_total_value
 
  def expiration_date_cannot_be_in_the_past
    if expiration_date.present? && expiration_date < Date.today
      errors.add(:expiration_date, "can't be in the past")
    end
  end
 
  def discount_cannot_be_greater_than_total_value
    if discount > total_value
      errors.add(:discount, "can't be greater than total value")
    end
  end
end
```

De forma predeterminada, dichas validaciones se ejecutarán cada vez que se llame `valid?`. También es posible controlar cuándo ejecutar estas validaciones personalizadas si se da una opción `:on` al método `validate`, con `:create` o `:update`

```ruby
class Invoice < ActiveRecord::Base
  validate :active_customer, on: :create
 
  def active_customer
    errors.add(:customer_id, "is not active") unless customer.active?
  end
end
```

















