# 3- Recibir correos electrónicos

Recibir y analizar correos electrónicos con Action Mailer puede ser un esfuerzo bastante complejo. Antes de que tu correo electrónico llegue a tu aplicación de Rails, tendrías que configurar tu sistema de alguna manera para enviar correos electrónicos a tu aplicación, que debe estar escuchando eso. Por lo tanto, para recibir correos electrónicos en tu aplicación de Rails necesitarás:

* Implementar un método de recepción en su mailer.
* Configure su servidor de correo electrónico para reenviar los correos electrónicos de la\(s\) dirección\(es\) a la que le gustaría recibir su aplicación a `/path/to/app/bin/rails runner 'UserMailer.receive (STDIN.read)'`

Una vez que el método `receieve` es llamado se define en cualquier mailer, Action Mailer analizará el correo electrónico entrante sin procesar en un objeto, lo decodifica, lo instancia en un nuevo mailer y pasa el objeto de mail al método de la instancia de `receieve` del cliente. He aquí un ejemplo:

```ruby
class UserMailer < ApplicationMailer
  def receive(email)
    page = Page.find_by(address: email.to.first)
    page.emails.create(
      subject: email.subject,
      body: email.body
    )
 
    if email.has_attachments?
      email.attachments.each do |attachment|
        page.attachments.create({
          file: attachment,
          description: email.subject
        })
      end
    end
  end
end
```



