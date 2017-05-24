# 8. Callbacks Condicionales

Así como las validaciones, podemos también crear la llamada de un método condicional sobre la satisfacción de un predicado dado. También podemos hacer esto utilizando las opciones `:if` y `:unless`, las cuales pueden tomar un símbolo, un string, un Proc o un Array. Puedes utilizar la opción `:if` cuando quieras especificar bajo que condiciones el callback debería ser llamado. Si quieres especificar las condiciones bajo las cuales el callback no debería ser llamado, podrías utilizar la opción `:unless`. 



### 8.1 Utilizando :if y :unless con un Símbolo 

Puedes asociar la opciones `:if` y `:unless` con un símbolo correspondiendo con el nombre del método predicado que será llamado correctamente antes del callback. Cuando usamos la opción `:if`, el callback no se ejecutará si el método predicado retorna `false`; cuando utilizamos la opción `:unless`, el callback no será ejecutado si el método predicado retorna `true`. Esta es la opción más común. Utilizando esta forma de registrarlos también es posible registrar varios predicados diferentes que deberían ser llamados para comprobar si el callback debería ser ejecutado.

```ruby
class Order < ActiveRecord::Base
  before_save :normalize_card_number, if: :paid_with_card?
end
```



### 8.2 Utilizando :if y :unless como un String 

También puedes utilizar un string que será evaluado que será evaluado utilizando `eval` y por lo tanto necesita contener código Ruby válido. Deberías utilizar esta opción solo cuando el string representa una condición realmente corta:

```ruby
class Order < ActiveRecord::Base
  before_save :normalize_card_number, if: "paid_with_card?"
end
```



### 8.3 Utilizando :if y :unless con un Proc 

Finalmente, esto es posible para asociar `:if` y `:unless` con un objeto Proc. Esta opción es la mejor combinación cuando escribimos métodos cortos para validar, normalmente una línea:

```ruby
class Order < ActiveRecord::Base
  before_save :normalize_card_number,
    if: Proc.new { |order| order.paid_with_card? }
end
```



### 8.4 Condiciones Multiples para Callbacks 

Cuando escribimos callbacks condicionales, podemos también mezclar ambos `:if` y `:unless` en la misma declaración callback:

```ruby
class Comment < ActiveRecord::Base
  after_create :send_email_to_author, if: :author_wants_emails?,
    unless: Proc.new { |comment| comment.article.ignore_comments? }
end
```







