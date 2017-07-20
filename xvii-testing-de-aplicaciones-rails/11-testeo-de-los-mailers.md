# 11- Testeo de los Mailers

El testeo de las clases de Mailer requiere algunas herramientas específicas para hacer un trabajo a fondo.

### 11.1 Mantener al Postman bajo control

Las clases de correo electrónico - como cualquier otra parte de su aplicación de Rails - deben ser probados para asegurarse de que están funcionando como se esperaba.

Los objetivos de probar sus clases de correo son asegurarse que:

* Los correos electrónicos están siendo procesados \(creados y enviados\)
* El contenido del correo electrónico es correcto \(sujeto, remitente, cuerpo, etc\)
* Los correos electrónicos correctos se envían en los momentos correctos

#### 11.1.1 Desde todos los lados

Hay dos aspectos en los test de su mailer, las pruebas unitarias y las pruebas funcionales. En las pruebas unitarias, ejecuta el mailer de forma aislada con entradas estrictamente controladas y compara la salida con un valor conocido \(un fixture\). En las pruebas funcionales no se ponen a prueba los detalles minuciosos producidos por el remitente; En su lugar, probamos que nuestros controladores y modelos están utilizando el correo electrónico de la manera correcta. Prueba para probar que el correo electrónico correcto fue enviado en el momento adecuado.

### 11.2 Pruebas unitarias

Con el fin de probar que su mailer está funcionando como se esperaba, puede utilizar pruebas unitarias para comparar los resultados reales del mailer con ejemplos preescritos de lo que debe producirse.

#### 11.2.1 Venganza de los fixtures

Para el propósito de hacer pruebas unitarias de un mailer, los fixtures se utilizan para proporcionar un ejemplo de cómo el resultado debe lucir. Debido a que estos son ejemplos de mensajes de correo electrónico, y no datos de Active Record como los otros fixtures, que se mantienen en su propio subdirectorio aparte de los otros fixtures. El nombre del directorio dentro de `test/fixtures` corresponde directamente al nombre del mailer. Por lo tanto, para un mailer llamado `UserMailer`, los fixtures deben residir en el directorio `test/fixtures/user_mailer.`

Cuando generó su mailer, el generador crea fixtures de stub para cada una de las acciones de los remitentes. Si no usó el generador, tendrá que crear esos archivos usted mismo.

#### 11.2.2 El caso de prueba básica

Aquí hay una prueba de unidad para probar un mailer llamado `UserMailer` cuya invitación de acción se utiliza para enviar una invitación a un amigo. Es una versión adaptada de la prueba base creada por el generador para una acción de invitación.

```ruby
require 'test_helper'

class UserMailerTest < ActionMailer::TestCase
  test "invite" do
    # Create the email and store it for further assertions
    email = UserMailer.create_invite('me@example.com',
                                     'friend@example.com', Time.now)

    # Send the email, then test that it got queued
    assert_emails 1 do
      email.deliver_now
    end

    # Test the body of the sent email contains what we expect it to
    assert_equal ['me@example.com'], email.from
    assert_equal ['friend@example.com'], email.to
    assert_equal 'You have been invited by me@example.com', email.subject
    assert_equal read_fixture('invite').join, email.body.to_s
  end
end
```

En la prueba enviamos el correo electrónico y almacenamos el objeto devuelto en la variable de correo electrónico. Entonces nos aseguramos de que fue enviado \(la primera aserción\), entonces, en el segundo lote de aserciones, nos aseguramos de que el correo electrónico sí contiene lo que esperamos. El ayudante `read_fixture` se utiliza para leer el contenido de este archivo.

`email.body.to_s` está presente cuando sólo hay una parte \(HTML o texto\) presente. Si el mailer proporciona ambos, puede probar su fixture contra partes específicas con `email.text_part.body.to_s` o `email.html_part.body.to_s`.

Aquí está el contenido del fixture de invitación:

```ruby
Hi friend@example.com,

You have been invited.

Cheers!
```

Este es el momento adecuado para entender un poco más sobre la escritura de pruebas para sus mailers. La línea `ActionMailer::Base.delivery_method = :test` en `config/environments/test.rb` establece el método `delivery` en modo de prueba para que el correo electrónico no se entregue realmente \(útil para evitar el spam de sus usuarios durante la prueba\), sino que se anexará a una matriz \(`ActionMailer::Base.deliveries`\).

> El array `ActionMailer::Base.deliveries` sólo se restablece automáticamente en las pruebas `ActionMailer::TestCase` y `ActionDispatch::IntegrationTest`. Si desea tener una pizarra limpia fuera de estos casos de prueba, puede restablecerla manualmente con: `ActionMailer::Base.deliveries.clear`

### 11.3 Pruebas funcionales

Las pruebas funcionales para mailers envuelve más que sólo comprobar que el cuerpo del correo electrónico, los destinatarios y así sucesivamente están correctas. En las pruebas de correo funcionales se llaman los métodos delivery de correo y se comprueba que los correos electrónicos apropiados se han agregado a la lista de entrega. Es bastante seguro suponer que los métodos de delivery hacen su trabajo. Usted está probablemente más interesado en si su propia lógica de negocio está enviando un email cuando usted espera que lo haga. Por ejemplo, puede comprobar que la operación invitar a un amigo está enviando un correo electrónico de la manera adecuada:

```ruby
require 'test_helper'
 
class UserControllerTest < ActionDispatch::IntegrationTest
  test "invite friend" do
    assert_difference 'ActionMailer::Base.deliveries.size', +1 do
      post invite_friend_url, params: { email: 'friend@example.com' }
    end
    invite_email = ActionMailer::Base.deliveries.last
 
    assert_equal "You have been invited by me@example.com", invite_email.subject
    assert_equal 'friend@example.com', invite_email.to[0]
    assert_match(/Hi friend@example.com/, invite_email.body.to_s)
  end
end
```









