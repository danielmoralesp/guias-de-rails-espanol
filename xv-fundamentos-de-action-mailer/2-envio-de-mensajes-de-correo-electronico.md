# 2- Envío de mensajes de correo electrónico

Esta sección proporcionará una guía paso a paso para crear un mailer y sus vistas.

### 2.1 Pasos a seguir para generar un Mailer

#### 2.1.1 Crear el Mailer

```ruby
$ bin/rails generate mailer UserMailer
create  app/mailers/user_mailer.rb
create  app/mailers/application_mailer.rb
invoke  erb
create    app/views/user_mailer
create    app/views/layouts/mailer.text.erb
create    app/views/layouts/mailer.html.erb
invoke  test_unit
create    test/mailers/user_mailer_test.rb
create    test/mailers/previews/user_mailer_preview.rb
```

```ruby
# app/mailers/application_mailer.rb
class ApplicationMailer < ActionMailer::Base
  default from: "from@example.com"
  layout 'mailer'
end

# app/mailers/user_mailer.rb
class UserMailer < ApplicationMailer
end
```

Como puede ver, puede generar correos electrónicos como si utilizara otros generadores con Rails. Los mailers son conceptualmente similares a los controladores, y así recibimos un mailer, un directorio para vistas y uno para pruebas.

Si no desea utilizar un generador, puede crear su propio archivo dentro de la `app/mailers`, sólo asegúrese de que hereda de `ActionMailer::Base`:

#### 2.1.2 Editar el Mailer

Los mailers son muy similares a los controladores de Rails. También tienen métodos llamados "acciones" y usan vistas para estructurar el contenido. Cuando un controlador genera contenido como HTML para enviar de vuelta al cliente, un Mailer crea un mensaje para ser entregado por correo electrónico.

`app/mailers/user_mailer.rb` contiene un correo vacío:

```ruby
class UserMailer < ApplicationMailer
end
```

Vamos a agregar un método llamado `welcome_email`, que enviará un correo electrónico a la dirección de correo electrónico del usuario:

```ruby
class UserMailer < ApplicationMailer
  default from: 'notifications@example.com'

  def welcome_email(user)
    @user = user
    @url  = 'http://example.com/login'
    mail(to: @user.email, subject: 'Welcome to My Awesome Site')
  end
end
```

Aquí hay una explicación rápida de los elementos presentados en el método anterior. Para obtener una lista completa de todas las opciones disponibles, por favor, eche un vistazo más abajo en la lista completa de Action Mailer user-settable atributos de sección.

* default Hash: este es un hash de valores predeterminados para cualquier correo electrónico que envíe desde este mailer. En este caso, lo estamos estableciendo desde `:from` en el encabezado hasta un valor para todos los mensajes de esta clase. Esto puede anularse por correo electrónico.
* mail - El mensaje de correo electrónico real, estamos pasandolo en los encabezados por medio de `:to` y `:subject`.

Al igual que los controladores, cualquier variable de instancia que definamos en el método estará disponible para su uso en las vistas.

#### 2.1.3 Crear una vista de correo

Cree un archivo llamado `welcome_email.html.erb` en `app/views/user_mailer/`. Esta será la plantilla utilizada para el correo electrónico, formateada en HTML:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta content='text/html; charset=UTF-8' http-equiv='Content-Type' />
  </head>
  <body>
    <h1>Welcome to example.com, <%= @user.name %></h1>
    <p>
      You have successfully signed up to example.com,
      your username is: <%= @user.login %>.<br>
    </p>
    <p>
      To login to the site, just follow this link: <%= @url %>.
    </p>
    <p>Thanks for joining and have a great day!</p>
  </body>
</html>
```

Hagamos también una parte de texto para este correo electrónico. No todos los clientes prefieren los correos electrónicos HTML, por lo que el envío de ambos es la mejor práctica. Para ello, cree un archivo denominado `welcome_email.text.erb` en `app/views/user_mailer/`:

```
Welcome to example.com, <%= @user.name %>
===============================================

You have successfully signed up to example.com,
your username is: <%= @user.login %>.

To login to the site, just follow this link: <%= @url %>.

Thanks for joining and have a great day!
```

Cuando llame al método de correo ahora, Action Mailer detectará las dos plantillas \(texto y HTML\) y generará automáticamente un correo electrónico de varias `multipart/alternative`.

#### 2.1.4 Llamar al Mailer

Los mailers son realmente sólo otra manera de hacer una vista. En lugar de representar una vista y enviarla a través del protocolo HTTP, simplemente la están enviando a través de los protocolos de correo electrónico. Debido a esto, tiene sentido que su controlador le diga al remitente que envíe un correo electrónico cuando un usuario se cree correctamente.

Establecer esto es dolorosamente simple.

En primer lugar, vamos a crear un scaffold de usuario simple:

```ruby
$ bin/rails generate scaffold user name email login
$ bin/rails db:migrate
```

Ahora que tenemos un modelo de usuario con el que jugar, solo editaremos `app/controllers/users_controller.rb` para que instruya al UserMailer a entregar un correo electrónico al usuario recién creado editando la acción create e insertando una llamada a `UserMailer.welcome_email` Justo después de que el usuario se guarde correctamente.

Action Mailer está muy bien integrado con Active Job para que pueda enviar correos electrónicos fuera del ciclo de solicitud y respuesta, por lo que el usuario no tiene que esperar:

```ruby
class UsersController < ApplicationController
  # POST /users
  # POST /users.json
  def create
    @user = User.new(params[:user])

    respond_to do |format|
      if @user.save
        # Tell the UserMailer to send a welcome email after save
        UserMailer.welcome_email(@user).deliver_later

        format.html { redirect_to(@user, notice: 'User was successfully created.') }
        format.json { render json: @user, status: :created, location: @user }
      else
        format.html { render action: 'new' }
        format.json { render json: @user.errors, status: :unprocessable_entity }
      end
    end
  end
end
```

> El comportamiento predeterminado de Active Job es ejecutar trabajos a través del `:async adapter`. Por lo tanto, puede utilizar `deliver_later` ahora para enviar mensajes de correo electrónico de forma asíncrona. El adaptador predeterminado de Active Job ejecuta trabajos con un grupo de subprocesos en proceso. Es muy adecuado para los entornos de development/test, ya que no requiere ninguna infraestructura externa, pero es poco adecuado para producción, ya que deja caer los Jobs pendientes al reiniciar. Si necesita un backend persistente, necesitará utilizar un adaptador de Active Job que tenga un backend persistente \(Sidekiq, Resque, etc.\).

Si quieres enviar correos electrónicos de inmediato \(de un cronjob por ejemplo\) sólo tienes que llamar a `deliver_now`:

```ruby
class SendWeeklySummary
  def run
    User.find_each do |user|
      UserMailer.weekly_summary(user).deliver_now
    end
  end
end
```

El método `welcome_email` devuelve un objeto `ActionMailer::MessageDelivery` que luego se le puede decir `deliver_now` o `deliver_later` para enviarse. El objeto `ActionMailer::MessageDelivery` es sólo un contenedor alrededor de un `Mail::Message`. Si desea inspeccionar, alterar o hacer cualquier otra cosa con el objeto `Mail::Message` puede acceder a él con el método de mensaje en el objeto `ActionMailer::MessageDelivery`.

### 2.2 Valores de encabezado de codificación automática

Action Mailer gestiona la codificación automática de caracteres multibyte dentro de los encabezados y los cuerpos.

Para ejemplos más complejos, como la definición de conjuntos de caracteres alternativos o el texto de autocodificación, consulte la biblioteca de [Mail](https://github.com/mikel/mail).

### 2.3 Lista completa de métodos de Action Mailer

Sólo hay tres métodos que necesita para enviar prácticamente cualquier mensaje de correo electrónico:

* `headers` - Especifica cualquier encabezado en el correo electrónico que desee. Puede pasar un hash de nombres de campo de encabezado y pares de valores, o puede llamar a `headers[:field_name] = 'value'`.
* `attachments`: le permite agregar archivos adjuntos a su correo electrónico. Por ejemplo, los archivos `attachments['file-name.jpg'] = File.read('file-name.jpg')`.
* `mail` - Envía el correo electrónico en sí mismo. Puede pasarlo en los headers como un hash al método de mail como un parámetro, el mail creará un correo electrónico, ya sea de texto plano o multipart, dependiendo de las plantillas de correo electrónico que haya definido.

#### 2.3.1 Adición de archivos adjuntos

Action Mailer hace que sea muy fácil agregar archivos adjuntos.

Pase el nombre y el contenido del archivo y Action Mailer y la [gema mail](https://github.com/mikel/mail) automáticamente adivinarán el tipo mime, establecerán la codificación y crearán el archivo adjunto.

```ruby
attachments['filename.jpg'] = File.read('/path/to/filename.jpg')
```

Cuando se active el método de mail, enviará un correo electrónico multipart con un archivo adjunto, correctamente anidado con el nivel superior que es `multipart/mixed`y la primera parte es una `multipart/alternative` que contiene el texto sin formato y mensajes de correo electrónico HTML.

> Mail codificará automáticamente un archivo adjunto. Si desea algo diferente, codifique su contenido y pase el contenido codificado y la codificación en un Hash al método de archivos adjuntos.

Pase el nombre del archivo y especifique los encabezados y el contenido, y Action Mailer and Mail usará la configuración que le pase.

```ruby
encoded_content = SpecialEncode(File.read('/path/to/filename.jpg'))
attachments['filename.jpg'] = {
  mime_type: 'application/x-gzip',
  encoding: 'SpecialEncoding',
  content: encoded_content
}
```

> Si especifica una codificación, Mail asumirá que su contenido ya está codificado y no intenta codificar en Base64.

#### 2.3.2 Creando archivos adjuntos inline

Action Mailer 3.0 crea adjuntos inline, lo que implicaba un montón de hacking en pre versiones 3.0, y ya es mucho más simple y trivial como deberían ser.

Primero, para decirle a Mail que convierta un archivo adjunto en un archivo adjunto inline, simplemente llame a `#inline` en el método de archivos adjuntos dentro de su Mailer:

```ruby
def welcome
  attachments.inline['image.jpg'] = File.read('/path/to/image.jpg')
end
```

A continuación, en sus vistas, sólo puede hacer referencia a los archivos adjuntos como un hash y especificar qué adjunto desea mostrar, llamando a la url y luego pasando el resultado al método `image_tag`:

```ruby
<p>Hello there, this is our image</p>

<%= image_tag attachments['image.jpg'].url %>
```

Como se trata de una llamada estándar a `image_tag`, puedes pasar un hash de opciones después de la URL del attachment como lo harías para cualquier otra imagen:

```ruby
<p>Hello there, this is our image</p>

<%= image_tag attachments['image.jpg'].url, alt: 'My Photo', class: 'photos' %>
```

#### 2.3.3 Envío de correo electrónico a varios destinatarios

Es posible enviar correo electrónico a uno o más destinatarios en un mismo email \(por ejemplo, informar a todos los administradores de una nueva suscripción\) estableciendo la lista de correos electrónicos en la llave `:to`. La lista de correos electrónicos puede ser una matriz de direcciones de correo electrónico o una sola cadena con las direcciones separadas por comas.

```ruby
class AdminMailer < ActionMailer::Base
  default to: Proc.new { Admin.pluck(:email) },
          from: 'notification@example.com'

  def new_registration(user)
    @user = user
    mail(subject: "New User Signup: #{@user.email}")
  end
end
```

El mismo formato se puede utilizar para establecer "Con copia" \(Cc :\) y oculto \(Bcc :\), utilizando las llaves `:cc` y `:bcc` respectivamente.

#### 2.3.4 Envío de correo electrónico con nombre

A veces desea mostrar el nombre de la persona en lugar de su dirección de correo electrónico cuando reciba el email. El truco para hacerlo es formatear la dirección de correo electrónico en el formato "`Full name <email>`".

```ruby
def welcome_email(user)
  @user = user
  email_with_name = %("#{@user.name}" <#{@user.email}>)
  mail(to: email_with_name, subject: 'Welcome to My Awesome Site')
end
```

### 2.4 Vistas de email

Las vistas de correo se encuentran en el directorio `app/views/name_of_mailer_class`. La vista de correo específico es reconocida por la clase porque su nombre es el mismo que el método de correo. En nuestro ejemplo de arriba, nuestra vista de correo para el método `welcome_email` estará en `app/views/user_mailer/welcome_email.html.erb` para la versión HTML y `welcome_email.text.erb` para la versión de texto sin formato.

Para cambiar la vista de correo predeterminada de su acción, haga algo como:

```ruby
class UserMailer < ApplicationMailer
  default from: 'notifications@example.com'

  def welcome_email(user)
    @user = user
    @url  = 'http://example.com/login'
    mail(to: @user.email,
         subject: 'Welcome to My Awesome Site',
         template_path: 'notifications',
         template_name: 'another')
  end
end
```

En este caso buscará plantillas en `app/views/notifications` con otro nombre. También puede especificar una matriz de rutas de `template_path`, y se buscarán en orden.

Si desea más flexibilidad, también puede pasar un bloque y renderizar plantillas específicas o incluso renderizar inline o texto sin utilizar un archivo de plantilla:

```ruby
class UserMailer < ApplicationMailer
  default from: 'notifications@example.com'

  def welcome_email(user)
    @user = user
    @url  = 'http://example.com/login'
    mail(to: @user.email,
         subject: 'Welcome to My Awesome Site') do |format|
      format.html { render 'another_template' }
      format.text { render text: 'Render text' }
    end
  end
end
```

Esto hará que la plantilla `'another_template.html.erb`' para la parte HTML y utilice el texto representado para la parte de texto. El comando `render` es el mismo utilizado dentro de Action Controller, por lo que puede utilizar todas las mismas opciones, tales como `:text` , `:inline` etc.

#### 2.4.1 Almacenamiento en caché de la vista de correo

Usted puede hacer la memoria caché en las vistas de correo como en las vistas de la aplicación utilizando el método `cache`.

```ruby
<% cache do %>
  <%= @company.name %>
<% end %>
```

Y para poder utilizar esta función, necesitas configurar tu aplicación con esto:

```ruby
config.action_mailer.perform_caching = true
```

### 2.5 Plantillas de Action Mailer

Al igual que las vistas del controlador, también puede tener layouts de correo. El nombre de layout debe ser el mismo que el de correo, como `user_mailer.html.erb` y `user_mailer.text.erb` para ser reconocido automáticamente por su mailer como un layout.

```ruby
class UserMailer < ApplicationMailer
  layout 'awesome' # use awesome.(html|text).erb as the layout
end
```

Al igual que con las vistas del controlador, utilice `yield` para representar la vista dentro del layout.

También puede pasar una opción de layout: '`layout_name`' a la llamada de `render` dentro del bloque de formato para especificar diferentes layouts para diferentes formatos:

```ruby
class UserMailer < ApplicationMailer
  def welcome_email(user)
    mail(to: user.email) do |format|
      format.html { render layout: 'my_layout' }
      format.text
    end
  end
end
```

Proporcionará la parte HTML utilizando el archivo `my_layout.html.erb` y la parte de texto con el archivo `user_mailer.text.erb` habitual si existe.

### 2.6 Vista previa de correos electrónicos

Las previsualizaciones de Action Mailer proporcionan una forma de ver cómo se ven los correos electrónicos visitando una URL especial que los procesa. En el ejemplo anterior, la clase de vista previa para `UserMailer` debe llamarse `UserMailerPreview` y se encuentra en `test/mailers/previews/user_mailer_preview.rb`. Para ver la vista previa de `welcome_email`, implemente un método que tenga el mismo nombre y llame a `UserMailer.welcome_email`:

```ruby
class UserMailerPreview < ActionMailer::Preview
  def welcome_email
    UserMailer.welcome_email(User.first)
  end
end
```

A continuación, la vista previa estará disponible en `http://localhost:3000/rails/mailers/user_mailer/welcome_email.`

Si cambia algo en `app/views/user_mailer/welcome_email.html.erb` o en el propio correo, se volverá a cargar automáticamente y renderizarlo para que visualice el nuevo estilo al instante. Una lista de vistas previas también está disponible en `http://localhost:3000/rails/mailers`

De forma predeterminada, estas clases de vista previa están en `test/mailers/previews`. Esto se puede configurar usando la opción `preview_path`. Por ejemplo, si desea cambiarlo a `lib/mailer_previews`, puede configurarlo en `config/application.rb`:

```ruby
config.action_mailer.preview_path = "#{Rails.root}/lib/mailer_previews"
```

### 2.7 Generación de URLs en las vistas de Action Mailer

A diferencia de los controladores, la instancia de mailer no tiene ningún contexto sobre la solicitud entrante por lo que tendrá que proporcionar el parámetro `:host` usted mismo.

A medida que el `:host` suele ser consistente en la aplicación, se puede configurar globalmente en `config/application.rb`:

```ruby
config.action_mailer.default_url_options = { host: 'example.com' }
```

Debido a este comportamiento no puede utilizar ninguno de los helpers `*_path`  dentro de un correo electrónico. En su lugar, necesitará utilizar el auxiliar asociado `*_url`. Por ejemplo, en lugar de usar

```ruby
<%= link_to 'welcome', welcome_path %>
```

Usted necesitará usar:

```ruby
<%= link_to 'welcome', welcome_url %>
```

Al usar la URL completa, sus vínculos funcionarán ahora en sus correos electrónicos.

#### 2.7.1 generando URLs con url\_for

Url\_for genera la URL completa por defecto en las plantillas.

Si no configuró la opción `:host` globalmente asegúrese de pasarla a `url_for`.

```ruby
<%= url_for(host: 'example.com',
            controller: 'welcome',
            action: 'greeting') %>
```

#### 2.7.2 Generación de URL con rutas con nombre

Los clientes de correo electrónico no tienen ningún contexto web y por lo tanto las rutas no tienen una URL de base para formar direcciones web completas. Por lo tanto, siempre debe utilizar la variante "`_url`" de ayudantes de ruta con nombre.

Si no configuró la opción `:host` globalmente, asegúrese de pasarla al ayudante de la url.

```ruby
<%= user_url(@user, host: 'example.com') %>
```

> Los enlaces no-GET requieren `rails-ujs` o `jQuery_UJS`, y no funcionarán en las plantillas de correo. Resultarán en peticiones GET normales.

### 2.8 Adición de imágenes en las vistas de Action Mailer

A diferencia de los controladores, la instancia de correo no tiene ningún contexto sobre la solicitud entrante por lo que tendrá que proporcionar el parámetro `:asset_host` usted mismo.

Como el `:asset_host` suele ser consistente en toda la aplicación, puede configurarlo globalmente en `config/application.rb`:

```ruby
config.action_mailer.asset_host = 'http://example.com'
```

Ahora puede mostrar una imagen dentro de su correo electrónico.

```ruby
<%= image_tag 'image.jpg' %>
```

### 2.9 Envío de correos electrónicos multipart

Action Mailer enviará automáticamente correos electrónicos multipart si tiene plantillas diferentes para la misma acción. Por lo tanto, para nuestro ejemplo de `UserMailer`, si tiene `welcome_email.text.erb` y `welcome_email.html.erb` en `app/views/user_mailer`, Action Mailer enviará automáticamente un correo electrónico multipart con las versiones de texto HTML y texto como multipart.

El orden de las piezas que se insertan es determinado por el `:parts_order` dentro del método `ActionMailer::Base.default`.

### 2.10 Envío de correos electrónicos con opciones dinámicas de entrega

Si desea reemplazar las opciones de entrega predeterminadas \(por ejemplo, las credenciales de SMTP\) durante la entrega de correos electrónicos, puede hacerlo utilizando `delivery_method_options` en la acción de correo.

```ruby
class UserMailer < ApplicationMailer
  def welcome_email(user, company)
    @user = user
    @url  = user_url(@user)
    delivery_options = { user_name: company.smtp_user,
                         password: company.smtp_password,
                         address: company.smtp_host }
    mail(to: @user.email,
         subject: "Please see the Terms and Conditions attached",
         delivery_method_options: delivery_options)
  end
end
```

### 2.11 Envío de correos electrónicos sin renderización de plantillas

Puede haber casos en los que desee omitir el paso de renderización de la plantilla y suministrar el cuerpo del correo como una cadena. Puede lograr esto usando la opción` :body`. En estos casos, no olvide añadir la opción `:content_type`. Rails predeterminará el `text/plain` de lo contrario.

```ruby
class UserMailer < ApplicationMailer
  def welcome_email(user, email_body)
    mail(to: user.email,
         body: email_body,
         content_type: "text/html",
         subject: "Already rendered!")
  end
end
```



