# 6- Configuración de Action Mailer

Las siguientes opciones de configuración se hacen mejor en uno de los archivos de entorno \(`environment.rb,` `production.rb`, etc ...\)

| Configuración | Descripción |
| :--- | :--- |
| `logger` | Genera información sobre la ejecución del correo si está disponible. Se puede establecer en nil para no registrar. Compatible con los registradores Logger y Log4r propios de Ruby.. |
| `smtp_settings` | Allows detailed configuration for`:smtp`delivery method:`:address`- Allows you to use a remote mail server. Just change it from its default`"localhost"`setting.`:port`- On the off chance that your mail server doesn't run on port 25, you can change it.`:domain`- If you need to specify a HELO domain, you can do it here.`:user_name`- If your mail server requires authentication, set the username in this setting.`:password`- If your mail server requires authentication, set the password in this setting.`:authentication`- If your mail server requires authentication, you need to specify the authentication type here. This is a symbol and one of`:plain`\(will send the password in the clear\),`:login`\(will send password Base64 encoded\) or`:cram_md5`\(combines a Challenge/Response mechanism to exchange information and a cryptographic Message Digest 5 algorithm to hash important information\)`:enable_starttls_auto`- Detects if STARTTLS is enabled in your SMTP server and starts to use it. Defaults to`true`.`:openssl_verify_mode`- When using TLS, you can set how OpenSSL checks the certificate. This is really useful if you need to validate a self-signed and/or a wildcard certificate. You can use the name of an OpenSSL verify constant \('none' or 'peer'\) or directly the constant \(`OpenSSL::SSL::VERIFY_NONE`or`OpenSSL::SSL::VERIFY_PEER`\). |
| `sendmail_settings` | Allows you to override options for the`:sendmail`delivery method.`:location`- The location of the sendmail executable. Defaults to`/usr/sbin/sendmail`.`:arguments`- The command line arguments to be passed to sendmail. Defaults to`-i`. |
| `raise_delivery_errors` | Whether or not errors should be raised if the email fails to be delivered. This only works if the external email server is configured for immediate delivery. |
| `delivery_method` | Defines a delivery method. Possible values are:`:smtp`\(default\), can be configured by using`config.action_mailer.smtp_settings`.`:sendmail`, can be configured by using`config.action_mailer.sendmail_settings`.`:file`: save emails to files; can be configured by using`config.action_mailer.file_settings`.`:test`: save emails to`ActionMailer::Base.deliveries`array.See[API docs](http://api.rubyonrails.org/v5.1.2/classes/ActionMailer/Base.html)for more info. |
| `perform_deliveries` | Determines whether deliveries are actually carried out when the`deliver`method is invoked on the Mail message. By default they are, but this can be turned off to help functional testing. |
| `deliveries` | Keeps an array of all the emails sent out through the Action Mailer with delivery\_method :test. Most useful for unit and functional testing. |
| `default_options` | Allows you to set default values for the`mail`method options \(`:from`,`:reply_to`, etc.\). |

Para obtener una descripción completa de las configuraciones posibles, consulte la publicación Configuración de Action Mailer que vimos arriba en la sección 3.10.

### 6.1 Ejemplo de configuración de Action Mailer

Un ejemplo sería agregar lo siguiente a su archivo apropiado de `config/environments/$RAILS_ENV.rb`:

```ruby
config.action_mailer.delivery_method = :sendmail
# Defaults to:
# config.action_mailer.sendmail_settings = {
#   location: '/usr/sbin/sendmail',
#   arguments: '-i'
# }
config.action_mailer.perform_deliveries = true
config.action_mailer.raise_delivery_errors = true
config.action_mailer.default_options = {from: 'no-reply@example.com'}
```

### 6.2 Configuración de Action Mailer para Gmail

Como Action Mailer ahora usa la gema Mail, esto se vuelve tan simple como añadir a tu archivo `config/environments/$RAILS_ENV.rb`:

```ruby
config.action_mailer.delivery_method = :smtp
config.action_mailer.smtp_settings = {
  address:              'smtp.gmail.com',
  port:                 587,
  domain:               'example.com',
  user_name:            '<username>',
  password:             '<password>',
  authentication:       'plain',
  enable_starttls_auto: true  }
```

Nota: A partir del 15 de julio de 2014, Google aumentó sus medidas de seguridad y ahora bloquea los intentos de las aplicaciones que considera poco seguras. Puedes cambiar la configuración de Gmail [aquí](https://myaccount.google.com/lesssecureapps?pli=1) para permitir los intentos. Si su cuenta de Gmail tiene habilitada la autenticación de dos factores, deberá establecer una contraseña de aplicación y usarla en lugar de su contraseña normal. Alternativamente, puede usar otro ESP para enviar correo electrónico reemplazando '`smtp.gmail.com`' con la dirección de su proveedor.





