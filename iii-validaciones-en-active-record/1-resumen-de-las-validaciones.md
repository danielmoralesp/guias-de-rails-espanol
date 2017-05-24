# 1. Resumen de las Validaciones

Aquí hay un ejemplo de una validación muy simple:

```ruby
class Person < ActiveRecord::Base
  validates :name, presence: true
end

Person.create(name: "John Doe").valid? # => true
Person.create(name: nil).valid? # => false
```

Como puedes ver, nuestra validación nos hace saber que nuestro objeto `Person` no es válido sin un atributo `name`. La segunda instancia de `Person` no será guardada en la base de datos.

 Antes de entrar en más detalles, hablaremos de como las validaciones quedan en el cuadro general de nuestra aplicación.



### 1.1 ¿Por qué Utilizamos Validaciones?

Las validaciones son utilizadas para asegurarse que solo datos válidos son guardados dentro de la base de datos. Por ejemplo, esto puede ser muy importante para que tu aplicación se asegure que todos los usuarios proveen una dirección de correo electrónico y una dirección postal válidas. Las validaciones del nivel del Modelo, son la mejor manera de asegurar que solo los datos válidos son guardados en la base de datos. Son independientes del motor de base de datos, no los pueden sobrepasar los usuarios finales, y son recomendables para probar y mantener. Rails los hace fácil de utilizar, provee métodos ayudantes construídos previamente para las necesidades más comunes, y te permite crear tus propias validaciones también.

Hay varias otras maneras de validar los datos antes de guardarlos en tu base de datos, incluyendo una restricciones nativas de la base de datos, validaciones del lado del cliente y validaciones en al nivel del controlador. Aquí un resumen de las mismas:

* Restricciones de la base de datos, y/o procedimientos almacenados que hacen el mecanismo de validación dependiente del motor de la base de datos y pueden hacer las pruebas y el mantenimiento más dificil. Sin embargo, si tu base de datos es utilizada por otras aplicaciones, puede ser una buena idea utilizar algunas restricciones en el nivel de base de datos. Adicionalmente, las validaciones en el nivel de base de datos, pueden mantener la seguridad en algunas cosas \(por ejemplo la unicidad en tablas muy grandes\) que son difíciles de implementar de otra manera. 
* Las validaciones del lado del cliente suelen ser utilizadas, pero son generalmente poco confiables si se utilizan solas. Si son implementadas utilizando JavaScript, pueden ser sobrepasadas si el JavaScript está quitado en el navegador del usuario. Sin embargo, si se combinan con otras técnicas, las validaciones del lado del cliente pueden ser una manera conveniente de proporcionar una respuesta inmediata a quienes utilicen tu página. 
* Las validaciones en el nivel del controlador pueden ser temporalmente utilizadas, pero frecuentemente se vuelven incomprensibles y difíciles de probar y mantener. Siempre que sea posible, es una buena idea mantener los controladores limpios, esto hará que sea un placer trabajar en tu aplicación a largo plazo.

Elígelas con certeza, en casos específicos. Esta es la opinión del equipo de Rails que las validaciones al nivel del modelo son las más apropiadas en la mayoría de las circunstancias.



### 1.2 ¿Cuando Ocurren las Validaciones?

Hay dos clases de objetos Active Record: aquellos que se corresponden a un registro en la base de datos y los que no. Cuando creas un objeto nuevo, por ejemplo usando el método `new`, ese objeto no pertenece a la base de datos aún. Una vez que llamas al método `save` el objeto será guardado en la tabla apropiada de la base de datos. Active Record utiliza el método de instancia `new_record?` para determinar si un objeto está ya en la base de datos o no. Considera la siguiente clase Active Record:

```ruby
class Person < ActiveRecord::Base
end
```

Podemos ver como trabaja mirando algunas salidas en rails console:

```ruby
$ bin/rails console
>> p = Person.new(name: "John Doe")
=> #<Person id: nil, name: "John Doe", created_at: nil, updated_at: nil>
>> p.new_record?
=> true
>> p.save
=> true
>> p.new_record?
=> false
```

Al crear y guardar un nuevo registro se enviará una operación `SQL INSERT` a la base de datos. En cambio al actualizar un registro existente enviaremos una operación `SQL UPDATE`. Las validaciones típicamente son ejecutadas antes que estos comandos se envíen a la base de datos. Si alguna validación falla, el objeto será marcado como inválido y Active Record no creará las operaciones `INSERT` o `UPDATE`. De lo contrario se guardaría un objeto inválido en la base de datos. Puedes elegir tener validaciones específicas ejecutándose cuando un objeto es creado, guardado o actualizado.

Hay muchas maneras para cambiar el estado de un objeto en la base de datos. Algunos métodos dispararán validaciones, pero otros no lo harán. Esto significa que es posible guardar un objeto en la base de datos en un estado inválido si no tienes cuidado. 

Los siguientes métodos disparan validaciones, y guardarán un objeto en la base de datos solo si el objeto es válido:

```ruby
create
create!
save
save!
update
update!
```

Las versiones bang \(ej: `save!`\) lanzan una excepción si el registro es inválido. Las versiones no-bang como `save` y `update` retornan `false`, y `create` solo retorna el objeto.



### 1.3 Saltando las Validaciones

Los siguientes métodos saltan validaciones, y guardarán el objeto en la base de datos sin tener en cuenta su validez. Deben ser utilizados con precaución.

```ruby
decrement!
decrement_counter
increment!
increment_counter
toggle!
touch
update_all
update_attribute
update_column
update_columns
update_counters
```

Nota que save también tiene la posibilidad de saltar validaciones si se le pasa de argumento `validate: false`. Esta técnica debe ser utilizada con precaución. 

`save(validate: false)`



### 1.4 valid? e invalid?

Para verificar si un objeto es o no válido, Rails utiliza el método `valid?`. También puedes utilizar este método por ti mismo. `valid?` dispara tus validaciones y retorna `true` si no fueron encontrados errores en el objeto, y `false` de lo contrario. Como puedes ver aquí:

```ruby
class Person < ActiveRecord::Base
  validates :name, presence: true
end
 
Person.create(name: "John Doe").valid? # => true
Person.create(name: nil).valid? # => false
```

Después de que Active Record ha ejecutado las validaciones, cualquier error encontrado puede ser conocido a través de la instancia del método `errors.messages`, el cual retorna una colección de errores. Por definición, un objeto es válido si la colección está vacía después de ejecutar las validaciones. 

> Nota que un objeto instanciado con `new` no reportará errores mientras solo sea instanciado aunque sea técnicamente inválido, porque las validaciones no serán ejecutadas cuando utilizas new.



```ruby
class Person < ActiveRecord::Base
  validates :name, presence: true
end
 
>> p = Person.new
# => #<Person id: nil, name: nil>
>> p.errors.messages
# => {}

>> p.valid?
# => false
>> p.errors.messages
# => {name:["can't be blank"]}
 
>> p = Person.create
# => #<Person id: nil, name: nil>
>> p.errors.messages
# => {name:["can't be blank"]}
 
>> p.save
# => false

>> p.save!
# => ActiveRecord::RecordInvalid: Validation failed: Name can't be blank
 
>> Person.create!
# => ActiveRecord::RecordInvalid: Validation failed: Name can't be blank
```

`invalid?` es simplemente la inversa de `valid?`. Dispara las validaciones, retornando `true` si cualquier error es encontrado en el objeto, y `false` en caso contrario.



### 1.5 errors\[\]

Para verificar si un atributo en particular de un objeto es válido, lo puedes verificar utilizando `errors[:attribute]`. Esto retornará un un array de todos los errores por `:attribute`. Si no hay errores en un atributo específico, es devuelto un array vacío. 

Este método es solo utilizado después de que las validaciones se han ejectuado, porque solo inspecciona la colección de errores y no disparan validaciones por si mismos. Esto es diferente desde el método ActiveRecord::Base\#invalid? que explicamos antes porque este no verifica la validez del objeto no tiene ese objetivo. Solo comprueba para ver si hay errores encontrados en un atributo individual del objeto.







