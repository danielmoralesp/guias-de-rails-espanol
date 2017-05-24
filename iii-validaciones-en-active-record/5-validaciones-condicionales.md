# 5. Validaciones Condicionales

A veces tendrá sentido validar un objeto sólo cuando se cumpla un predicado dado. Puede hacerlo usando las opciones `:if` y `:unless`, que pueden tomar un símbolo, una cadena, un Proc o un Array. Puede utilizar la opción `:if` si desea especificar cuándo debe ocurrir la validación. Si desea especificar cuándo no debería ocurrir la validación, puede utilizar la opción `:unless`. 



### 5.1 Uso de un símbolo :if y :unless 

Puede asociar las opciones `:if` y `:unless` en las opciones que tengan un símbolo que corresponda al nombre de un método que se llamará justo antes de que ocurra la validación. Esta es la opción más comúnmente utilizada.

```ruby
class Order < ActiveRecord::Base
  validates :card_number, presence: true, if: :paid_with_card?
 
  def paid_with_card?
    payment_type == "card"
  end
end
```



### 5.2 Uso de una cadena con :if y :unless 

También puede utilizar una cadena que se evaluará mediante `eval` y debe contener código Ruby válido. Debe utilizar esta opción sólo cuando la cadena representa una condición muy corta.

```ruby
class Person < ActiveRecord::Base
  validates :surname, presence: true, if: "name.nil?"
end
```



### 5.3 Usando un Proc con :if y :unless 

Por último, es posible asociar `:if` y `:unless` que con un objeto Proc que se llamará. El uso de un objeto Proc le da la posibilidad de escribir una condición en línea en lugar de un método separado. Esta opción es la más adecuada para los "one-liners".

```ruby
class Account < ActiveRecord::Base
  validates :password, confirmation: true,
    unless: Proc.new { |a| a.password.blank? }
end
```



### 5.4 Agrupación de Validaciones condicionales 

A veces es útil tener varias validaciones a usar en una condición, se puede lograr fácilmente usando `with_options`.

```ruby
class User < ActiveRecord::Base
  with_options if: :is_admin? do |admin|
    admin.validates :password, length: { minimum: 10 }
    admin.validates :email, presence: true
  end
end
```

Todas las validaciones dentro del bloque `with_options` habrán pasado automáticamente la condición `if: :is_admin?`



### 5.5 Combinación de condiciones de validación 

Por otro lado, cuando múltiples condiciones definen si una validación debe ocurrir o no, puede utilizarse un Array. Además, puede aplicar ambos `:if` y `:unless` a la misma validación.

```ruby
class Computer < ActiveRecord::Base
  validates :mouse, presence: true,
                    if: ["market.retail?", :desktop?],
                    unless: Proc.new { |c| c.trackpad.present? }
end
```

La validación sólo se ejecuta cuando todas las condiciones: `if` y ninguno de los `:unless` se evalúen a `true`.





