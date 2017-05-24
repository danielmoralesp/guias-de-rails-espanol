# 2. Helpers de Validación

Active Record ofrece muchos helpers de validación predefinidos que puedes utilizar directamente dentro de las definiciones de tu clase. Estos helpers proveen las reglas de validación comunes. Cada vez que una validación falla, un mensaje de error es añadido a la colección de `errors` del objeto, y este mensaje es asociado con el atributo que está siendo validado. 

Cada ayudante acepta un número arbitrario de nombres de atributos, entonces con una simple línea de código puedes añadir algún tipo de validadación a varios atributos. 

Todos ellos aceptan las opciones `:on` y `:message`, las cuales definen cuando la validación se ejecuta y el mensaje que debería ser añadido a la colección `errors` si falla, respectivamente. La opción `:on`  toma uno de los valores `:create` o `:update`. Hay un mensaje de error por defecto para cada uno de los helpers de validación. Estos mensajes son utilizados cuando la opción `:message` no está especificada. Vamos a repasar cada uno de los helpers disponibles.



### 2.1 acceptance

Este método valida si un checkbox en la interfaz de usuarios es chequeado al enviar un formulario. Esto es típicamente utilizado cuando el usuario necesita dar su conformidad sobre los términos y servicios de tu aplicación, confirmándolo después de leer algún texto o algo por el estilo. Esta validación es muy específica para aplicaciones web y este método `acceptance` no necesita guardarse en ningún sitio de tu base de datos \(si no tienes un campo para esto, el helper creará un atributo virtual\).

```ruby
class Person < ActiveRecord::Base
  validates :terms_of_service, acceptance: true
end
```

El mensaje de error por defecto de este helper es "must be accepted". 

Puede recibir una opción `:accept`, la cual determina que valor será considerado aceptable. Por defecto es "1" y puede ser cambiado fácilmente.

```ruby
class Person < ActiveRecord::Base
  validates :terms_of_service, acceptance: { accept: 'yes' }
end
```



### 2.2 validates\_associated

Deberías utilizar este helper cuando tu modelo tiene asociaciones con otros modelos y también necesiten ser validados. Cuando intentas guardar tu objeto `valid?` será llamado por cada uno de los objetos asociados.

```ruby
class Library < ActiveRecord::Base
  has_many :books
  validates_associated :books
end
```

Esta validación trabajará con todos los tipos de asociaciones. 

No utilices `validates_associated` en ambas puntas de tus asociaciones, porque se llamarán la una a la otra en un ciclo infinito. 

El mensaje de error por defecto para `validates_associated` es "is invalid". Nota que cada objeto asociado contendrá sus propias colecciones `errors`; los errores no escalan hacia el modelo llamado.



### 2.3 confirmation

Puedes utilizar este helper cuando tengas dos campos de texto que deban recibir exactamente el mismo contenido. Por ejemplo, puedes querer confirmar una dirección de email y un password. Esta validación crea un atributo virtual cuyo nombre es el campo que tiene que ser confirmado con la `_confirmation` añadida.

```ruby
class Person < ActiveRecord::Base
  validates :email, confirmation: true
end
```

En la plantilla de la vista podrías utilizar algo como

```ruby
<%= text_field :person, :email %>
<%= text_field :person, :email_confirmation %>
```

Esta comprobación se lleva a cabo solo si `email_confirmation` no es `nil`. Para requerir confirmación, asegúrate de añadir una comprobación `presence` al atributo de confirmación \(veremos `presence` más adelante en esta guía\):

```ruby
class Person < ActiveRecord::Base
  validates :email, confirmation: true
  validates :email_confirmation, presence: true
end
```

El mensaje de error para este ayudante es "doesn't match confirmation".



### 2.4 exclusion

Este helper valida que los valores de los atributos no están incluidos en una configuración dada. En realidad, esta configuración puede ser cualquier objeto enumerable.

```ruby
class Account < ActiveRecord::Base
  validates :subdomain, exclusion: { in: %w(www us ca jp),
    message: "%{value} is reserved." }
end
```

El helper `exclusion` tiene una opción `:in` que recibe la configuración de los valores que no serán aceptados por los atributos validados. La opción `:in` tiene un alias llamado `:within` que puedes utilizar para el mismo propósito, si lo quieres. Este ejemplo utiliza la opción `:message` para mostrar como puedes incluir los valores de los atributos. 

El mensaje de error por defecto es "is reserved".



### 2.5 format

Este helper valida los valores de los atributos por pruebas si coinciden o no con una expresión regular dada, que se especifica utilizando la opción `:with`.

```ruby
class Product < ActiveRecord::Base
  validates :legacy_code, format: { with: /\A[a-zA-Z]+\z/,
    message: "only allows letters" }
end
```

Alternativamente, puedes requerir que el atributo específico no responda a la expresión regular utilizando la opción `:without`. 

El mensaje de error por defecto es "is invalid".



### 2.6 inclusion

Este helper valida que los valores de los atributos estén incluidos en una configuración dada. En realidad, esta configuración puede ser un objeto enumerable.

```ruby
class Coffee < ActiveRecord::Base
  validates :size, inclusion: { in: %w(small medium large),
    message: "%{value} is not a valid size" }
end
```

El helper `inclusion` tiene una opción `:in` que recibe lo que debe ser incluido. La opción `:in` tiene un alias llamado `:within` que puedes utilizar para algunos propósitos, si lo deseas. Los ejemplos anteriores utilizan la opción `:message` para mostrar como puedes incluir valores de los atributos. 

El mensaje de error para este helper es "is not included in the list".



### 2.7 length

Este helper valida la longitud de los valores de los atributos. Provee una variedad de opciones, entonces puedes especificar restricciones de longitud de diferentes maneras:

```ruby
class Person < ActiveRecord::Base
  validates :name, length: { minimum: 2 }
  validates :bio, length: { maximum: 500 }
  validates :password, length: { in: 6..20 }
  validates :registration_number, length: { is: 6 }
end
```

Las opciones de restricción de longitud son: 

* `:minimum` - El atributo no puede tener menos caracteres que una longitud específica. 
* `:maximum` - El atributo no puede tener más caracteres que una longitud específica. 
* `:in` \(o `:within`\) - La longitud del atributo debe estar contenida en un intervalo dado. El valor para esta opción de ser un rango. 
* `:is` - La longitud del atributo debe ser igual a un valor dado.



Los mensajes de error por defecto dependen del tipo de validación de longitud que es utilizada. Puedes personalizar estos mensajes utilizando las opciones `:wrong_length`, `:too_long`, `:too_short` y `%{count}` como referencia para mostrar el número que corresponda para la restricción que se está utilizando. También puedes utilizar la opción `:message` para especificar un mensaje de error.

```ruby
class Person < ActiveRecord::Base
  validates :bio, length: { maximum: 1000,
    too_long: "%{count} characters is the maximum allowed" }
end
```

Este helper cuenta los caracteres por defecto, pero puedes separar el valor de diferentes maneras utilizanod la opción `:tokenizer option`:

```ruby
class Essay < ActiveRecord::Base
  validates :content, length: {
    minimum: 300,
    maximum: 400,
    tokenizer: lambda { |str| str.split(/\s+/) },
    too_short: "must have at least %{count} words",
    too_long: "must have at most %{count} words"
  }
end
```

> Nota que los mensajes de error por defecto están en plural \(ej: "is too short \(minimum is `%{count}` characters\)"\). Por esta razón, cuando `:minimum` es 1 debes indicar un mensaje personalizado o utilizar en su lugar `presence: true`. Cuando `:in` o `:within` tienen un un límite mínimo de 1, puedes proveer también un mensaje personalizado o llamar a `presence` antes que a `length`.



### 2.8 numericality

Este helper valida que tus atributos tienen solo valores numéricos. Por defecto, esto se comparará con una firma seguida por un entero o un número de punto flotante. Para especificar que solo números enteros están permitidos configuras a `true` `:only_integer`. 

Si configuras `:only_integer` a `true`, luego se utilizará la expresión regular `/\A[+-]?\d+\Z/ `para validar el valor del atributo. De otro modo, intentará convertir el valor a un número utilizando la expresión `Float`. 

> Nota que la expresión regular de arriba permite un caracter perdido de salto de línea.



```ruby
class Player < ActiveRecord::Base
  validates :points, numericality: true
  validates :games_played, numericality: { only_integer: true }
end
```

Además :only\_integer, este helper también acepta las siguientes opciones para añadir restricciones a los valores acceptables: 

* `:greater_than` - Especifica que el valor del atributo debe debe ser mayor que un valor determinado. El mensaje de error para esta opción es "must be greater than %{count}".
*  `:greater_than_or_equal_to` - Especifica que el valor del atributo debe ser mayor o igual a un valor determinado. El mensaje de error por defecto para esta opción es "must be greater than or equal to %{count}". 
* `:equal_to` - Especifica que el valor del atributo debe ser igual a un valor determinado. El mensaje de error por defecto de esta opción es "must be equal to %{count}".
* `:less_than` - Especifica que el valor del atributo debe ser menor que un determinado valor. El mensaje de error de esta opción es "must be less than %{count}". 
* `:less_than_or_equal_to` - Especifica que el valor del atributo debe ser menor o igual de un valor determinado. El mensaje de error por defecto es "must be less than or equal to %{count}". 
* `:odd` - Especifica que el valor debe ser un número impar si es configurado a `true`. El mensaje de error por defecto para esta opción es "must be odd".
* `:even` - Especifica que el valor del atributo debe ser un número par si es configurado a `true`. El mensaje de error por defecto para esta opción es "must be even".

Por defecto, `numericality` no permite valores `nil`. Puedes utilizar la opción `allow_nil: true` para permitirlo. 

El mensaje de error por defecto en este caso es "is not a number".



### 2.9 presence

Este ayudante valida que los atributos especificados no estén vacíos. Utiliza el método `blank?` para comprobar si el valor es `nil` o una cadena esta en blanco, es decir, una cadena que está vacía o consta de espacios en blanco.

```ruby
class Person < ActiveRecord::Base
  validates :name, :login, :email, presence: true
end
```

Si desea estar seguro de que una asociación está presente, deberá probar si el objeto asociado está presente y no la clave foránea utilizada para asignar la asociación.

```ruby
class LineItem < ActiveRecord::Base
  belongs_to :order
  validates :order, presence: true
end
```

En el modelo `order`, para validar los registros asociados cuya presencia es necesaria, debe especificar la opción `:inverse_of` para la asociación:

```ruby
class Order < ActiveRecord::Base
  has_many :line_items, inverse_of: :order
end
```

Si valida la presencia de un objeto asociado a través de una relación `has_one` o `has_many`, comprobará que el objeto no está en `blank?` Ni `marked_for_destruction?` 

Desde `false.blank?` Es `true`, si desea validar la presencia de un campo booleano debe utilizar una de las siguientes validaciones:

```ruby
validates :boolean_field_name, presence: true
validates :boolean_field_name, inclusion: { in: [true, false] }
validates :boolean_field_name, exclusion: { in: [nil] }
```

Mediante el uso de una de estas validaciones, se asegurará de que el valor NO será `nil` lo que resultaría en un valor `NULL` en la mayoría de los casos.



### 2.10 absence

Este ayudante valida que los atributos especificados están ausentes. Utiliza el método `present?` para comprobar si el valor no es `nil` o una cadena en blanco, es decir, una cadena que está vacía o consta de espacios en blanco.

```ruby
class Person < ActiveRecord::Base
  validates :name, :login, :email, absence: true
end
```

Si desea estar seguro de que una asociación está ausente, deberá probar si el objeto asociado está ausente y no la clave foránea utilizada para asignar la asociación.

```ruby
class LineItem < ActiveRecord::Base
  belongs_to :order
  validates :order, absence: true
end
```

Para validar los registros asociados cuyo `absence` es necesario, debe especificar la opción: `inverse_of` para la asociación:

```ruby
class Order < ActiveRecord::Base
  has_many :line_items, inverse_of: :order
end
```

Si valida la ausencia de un objeto asociado a través de una relación `has_one` o `has_many`, comprobará que el objeto no está `present?` Ni `marked_for_destruction?`. 

Desde `false.present?` Es falso, si desea validar la ausencia de un campo booleano que debe utilizar un validador `:field_name`, `exclusion: { in: [true, false] }`. 

El mensaje de error predeterminado es "must be blank".



### 2.11 uniqueness

Este ayudante valida que el valor del atributo es único justo antes de que el objeto sea guardado. No crea una restricción de unicidad en la base de datos, por lo que puede suceder que dos conexiones de base de datos diferentes creen dos registros con el mismo valor para una columna que desea ser único. Para evitarlo, debe crear un índice único en ambas columnas de la base de datos. Consulte el manual de MySQL para obtener más detalles sobre varios índices de columnas.

```ruby
class Account < ActiveRecord::Base
  validates :email, uniqueness: true
end
```

La validación ocurre realizando una consulta SQL en la tabla del modelo, buscando un registro existente con el mismo valor en ese atributo. 

Hay una opción de `:scope` que puede utilizar para especificar otros atributos que se utilizan para limitar la comprobación de unicidad:

```ruby
class Holiday < ActiveRecord::Base
  validates :name, uniqueness: { scope: :year,
    message: "should happen once per year" }
end
```

También hay una opción: `case_sensitive` que puede utilizar para definir si la restricción de unicidad será sensible a mayúsculas o minúsculas. Esta opción predeterminada es `true`.

```ruby
class Person < ActiveRecord::Base
  validates :name, uniqueness: { case_sensitive: false }
end
```

Tenga en cuenta que algunas bases de datos están configuradas para realizar búsquedas sin distinción entre mayúsculas y minúsculas. 

El mensaje de error predeterminado es "has already been taken".



### 2.12 validates\_with

Este ayudante pasa el registro a una clase separada para su validación.

```ruby
class GoodnessValidator < ActiveModel::Validator
  def validate(record)
    if record.first_name == "Evil"
      record.errors[:base] << "This person is evil"
    end
  end
end
 
class Person < ActiveRecord::Base
  validates_with GoodnessValidator
end
```

Los errores añadidos a `record.errors [:base]` se refieren al estado del registro como un todo y no a un atributo específico. 

El ayudante `validates_with` toma una clase, o una lista de clases para utilizar para la validación. No hay un mensaje de error predeterminado para `validates_with`. Debe agregar manualmente errores a la colección de errores del registro en la clase `validator`. 

Para implementar el método `validate`, debe tener definido un parámetro `record`, que es el registro a validar. 

Como todas las demás validaciones, `validates_with` toma las opciones `:if` ,`:unless` y `:on`. Si pasa otras opciones, enviará las opciones a la clase validator como `option`:

```ruby
class GoodnessValidator < ActiveModel::Validator
  def validate(record)
    if options[:fields].any?{|field| record.send(field) == "Evil" }
      record.errors[:base] << "This person is evil"
    end
  end
end
 
class Person < ActiveRecord::Base
  validates_with GoodnessValidator, fields: [:first_name, :last_name]
end
```

Tenga en cuenta que el validador se inicializará sólo una vez para todo el ciclo de vida de la aplicación, y no en cada ejecución de validación, así que tenga cuidado al usar variables de instancia dentro de ella. 

Si su validador es lo suficientemente complejo como para que desee variables de instancia, puede utilizar fácilmente un objeto Ruby antiguo



### 2.13 validates\_each

Este ayudante valida atributos contra un bloque. No tiene una función de validación predefinida. Deberías crear uno usando un bloque, y cada atributo pasado a `validates_each` será probado. En el ejemplo siguiente, no queremos que los nombres y apellidos comiencen con minúsculas.

```ruby
class Person < ActiveRecord::Base
  validates_each :name, :surname do |record, attr, value|
    record.errors.add(attr, 'must start with upper case') if value =~ /\A[[:lower:]]/
  end
end
```

El bloque recibe el registro, el nombre del atributo y el valor del atributo. Puede hacer cualquier cosa que desee para buscar datos válidos dentro del bloque. Si la validación falla, debe agregar un mensaje de error al modelo, por lo que no es válido.

















