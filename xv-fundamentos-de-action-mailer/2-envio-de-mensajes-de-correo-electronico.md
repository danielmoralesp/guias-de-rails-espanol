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

`headers` - Especifica cualquier encabezado en el correo electrónico que desee. Puede pasar un hash de nombres de campo de encabezado y pares de valores, o puede llamar a `headers[:field_name] = 'value'`.

`attachments`: le permite agregar archivos adjuntos a su correo electrónico. Por ejemplo, los archivos `attachments['file-name.jpg'] = File.read('file-name.jpg')`.







