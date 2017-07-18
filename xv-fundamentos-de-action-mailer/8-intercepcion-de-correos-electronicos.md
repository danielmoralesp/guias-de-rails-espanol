# 8- Intercepción de correos electrónicos

Hay situaciones en las que es necesario editar un correo electrónico antes de que sea entregado. Afortunadamente Action Mailer proporciona hooks para interceptar cada correo electrónico. Puede registrar un interceptor para realizar modificaciones en los mensajes de correo antes de entregarlos a los agentes de entrega.

```ruby
class SandboxEmailInterceptor
  def self.delivering_email(message)
    message.to = ['sandbox@example.com']
  end
end
```

Antes de que el interceptor pueda hacer su trabajo, necesita registrarlo con el framework de Action Mailer. Puede hacerlo en un archivo de inicialización` config/initializers/sandbox_email_interceptor.rb`

```ruby
if Rails.env.staging?
  ActionMailer::Base.register_interceptor(SandboxEmailInterceptor)
end
```

El ejemplo anterior utiliza un entorno personalizado denominado "staging" para un servidor de producción, pero con fines de prueba. Puede leer la sección Creación de entornos de Rails para obtener más información sobre los entornos personalizados de Rails.

