# 4. Validaciones Estrictas

También puede especificar validaciones para que sean estrictas y ejecutar `ActiveModel::StrictValidationFailed` cuando el objeto no es válido.

```ruby
class Person < ActiveRecord::Base
  validates :name, presence: { strict: true }
end
 
Person.new.valid?  # => ActiveModel::StrictValidationFailed: Name can't be blank
```

También existe la posibilidad de pasar una excepción personalizada a la opción `:strict`.

```ruby
class Person < ActiveRecord::Base
  validates :token, presence: true, uniqueness: true, strict: TokenGenerationException
end
 
Person.new.valid?  # => TokenGenerationException: Token can't be blank
```











