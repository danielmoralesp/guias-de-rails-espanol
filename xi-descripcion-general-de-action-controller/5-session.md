# 5- Session

Su aplicación tiene una sesión para cada usuario en la que puede almacenar pequeñas cantidades de datos que se mantendrán entre solicitudes. La sesión sólo está disponible en el controlador y la vista y puede utilizar uno de varios mecanismos de almacenamiento diferentes:

* `ActionDispatch::Session::CookieStore` - Almacena todo en el cliente.
* `ActionDispatch::Session::CacheStore` - Almacena los datos en la caché de Rails.
* `ActionDispatch::Session::ActiveRecordStore` - Almacena los datos en una base de datos utilizando Active Record. \(se requiere la gema `activerecord-session_store`\).
* `ActionDispatch::Session::MemCacheStore` - Almacena los datos en un clúster de memcached \(esto es una implementación heredada; considere usar CacheStore en su lugar\).

Todo el almacenamiento de las sesiones utiliza una cookie para almacenar un `ID` exclusivo para cada sesión \(debe usar una cookie, Rails no le permitirá pasar el `ID` de sesión en la URL, ya que es poco seguro\).

Para la mayoría del almacenamiento, este `ID` se utiliza para buscar los datos de sesión en el servidor, por ejemplo: en una tabla de base de datos. Hay una excepción, que es el almacenamiento de sesión predeterminado y recomendado - el `CookieStore` - que almacena todos los datos de sesión en la propia cookie \(el `ID` aún está disponible para usted si lo necesita\). Esto tiene la ventaja de ser muy ligero y requiere cero configuración en una nueva aplicación para poder usar la sesión. Los datos de las cookies se firman criptográficamente para que sean inviolables. Y también está encriptada para que cualquiera con acceso a ella no pueda leer su contenido. \(Rails no lo aceptará si se ha editado\).

El CookieStore puede almacenar alrededor de 4kB de datos - mucho menos que los otros - pero esto es generalmente suficiente. El almacenamiento de grandes cantidades de datos en la sesión se desaconseja sin importar el almacén de sesión que utilice su aplicación. Debe evitar especialmente el almacenamiento de objetos complejos \(cualquier objeto que no sea básico de Ruby, el ejemplo más común es instancias de un modelo\) en la sesión, ya que es posible que el servidor no pueda volver a montarlos entre las solicitudes, lo que resultará en un error.

Si sus sesiones de usuario no almacenan datos críticos o no necesitan estar presentes durante períodos largos \(por ejemplo, si sólo utiliza flash para mensajería\), puede considerar el uso de `ActionDispatch::Session::CacheStore`. Esto almacenará sesiones utilizando la implementación de caché que haya configurado para su aplicación. La ventaja de esto es que puede utilizar la infraestructura de caché existente para almacenar sesiones sin necesidad de ninguna configuración o administración adicional. La desventaja, por supuesto, es que las sesiones serán efímeras y podrían desaparecer en cualquier momento.

Obtenga más información sobre el almacenamiento de sesiones en la Guía de Seguridad.

Si necesita un mecanismo de almacenamiento de sesión diferente, puede cambiarlo en el inicializador:

```ruby
# Utilice la base de datos para las sesiones en lugar del predeterminado basado en cookies,
# que no debe usarse para almacenar información altamente confidencial
# (create the session table with "rails g active_record:session_migration")
# Rails.application.config.session_store :active_record_store
```

Rails configura una clave de sesión \(el nombre de la cookie\) al firmar los datos de la sesión. Estos también se pueden cambiar en un inicializador:

```ruby
# Be sure to restart your server when you modify this file.
Rails.application.config.session_store :cookie_store, key: '_your_app_session'
```

También puede pasar una clave de dominio y especificar el nombre de dominio para la cookie:

```ruby
# Be sure to restart your server when you modify this file.
Rails.application.config.session_store :cookie_store, key: '_your_app_session', domain: ".example.com"
```

Rails configura \(para CookieStore\) una clave secreta utilizada para firmar los datos de la sesión. Esto se puede cambiar en `config/secrets.yml`

```ruby
# Asegúrese de reiniciar su servidor cuando modifique este archivo.
# Su clave secreta se utiliza para verificar la integridad de las cookies firmadas.

# ¡Si cambia esta clave, todas las viejas cookies firmadas serán inválidas!
# asegúrese de que la clave es de al menos 30 caracteres y todo aleatorio,
# y no hay palabras regulares o estarás expuesto a ataques de diccionario.

# Puede utilizar `rails secret` para generar una clave secreta segura.
# asegúrese de que las claves de este archivo se mantengan privadas
# si está compartiendo su código públicamente.
 
development:
  secret_key_base: a75d...
 
test:
  secret_key_base: 492f...
 
# Do not keep production secrets in the repository,
# instead read values from the environment.
production:
  secret_key_base: <%= ENV["SECRET_KEY_BASE"] %>
```

> Cambiar la `secret` al usar CookieStore invalidará todas las sesiones existentes.

### 5.1 Acceso a la sesión

En su controlador puede acceder a la sesión a través del método de instancia de sesión.

> Las sesiones son de carga perezosa. Si no accede a las sesiones en el código de la acción, no se cargarán. Por lo tanto, nunca tendrá que desactivar las sesiones, simplemente no acceder a ellos hará el trabajo.

Los valores de sesión se almacenan utilizando pares clave/valor como un hash:

```ruby
class ApplicationController < ActionController::Base
 
  private
 
  # Encuentra el usuario con el ID almacenado en la sesión con la clave 
  # :current_user_id Esta es una forma común de manejar el inicio de sesión de un usuario en
  # una aplicación Rails; El inicio de sesión establece el valor de la sesión y
  # el cierre de sesión lo elimina.
  def current_user
    @_current_user ||= session[:current_user_id] &&
      User.find_by(id: session[:current_user_id])
  end
end
```

Para almacenar algo en la sesión, sólo tiene que asignarla a la clave como un hash:

```ruby
class LoginsController < ApplicationController
  # "Create" a login, aka "log the user in"
  def create
    if user = User.authenticate(params[:username], params[:password])
      # Save the user ID in the session so it can be used in
      # subsequent requests
      session[:current_user_id] = user.id
      redirect_to root_url
    end
  end
end
```

Para quitar algo de la sesión, asigne esa clave a `nil`:

```ruby
class LoginsController < ApplicationController
  # "Delete" a login, aka "log the user out"
  def destroy
    # Remove the user id from the session
    @_current_user = session[:current_user_id] = nil
    redirect_to root_url
  end
end
```

Para restablecer la sesión completa, use `reset_session`.

### 5.2 El Flash

El flash es una parte especial de la sesión que se borra con cada solicitud. Esto significa que los valores almacenados allí solo estarán disponibles en la siguiente petición, que es útil para pasar mensajes de error, etc.

Se accede de la misma manera que la sesión, como un hash \(es una instancia de FlashHash\).

Utilicemos el acto de cerrar sesión como ejemplo. El controlador puede enviar un mensaje que se mostrará al usuario en la siguiente solicitud:

```ruby
class LoginsController < ApplicationController
  def destroy
    session[:current_user_id] = nil
    flash[:notice] = "You have successfully logged out."
    redirect_to root_url
  end
end
```

Tenga en cuenta que también es posible asignar un mensaje de flash como parte de una redirección. Usted puede asignarlo como `:notice`, `:alert` o de propósito general `:flash`:

```ruby
redirect_to root_url, notice: "You have successfully logged out."
redirect_to root_url, alert: "You're stuck here!"
redirect_to root_url, flash: { referral_code: 1234 }
```

La acción `destroy` vuelve a dirigir al `root_url` de la aplicación, donde se mostrará el mensaje. Tenga en cuenta que hasta que no se ejecuta la siguiente acción no disparará el flash. Es una convención mostrar cualquier alerta o aviso de error de flash en el diseño de la aplicación:

```html
<html>
  <!-- <head/> -->
  <body>
    <% flash.each do |name, msg| -%>
      <%= content_tag :div, msg, class: name %>
    <% end -%>
 
    <!-- more content -->
  </body>
</html>
```

De esta manera, si una acción establece un aviso o un mensaje de alerta, el layout lo mostrará automáticamente.

Puede pasar cualquier cosa que la sesión pueda almacenar; No se limita a avisos y alertas:

```ruby
<% if flash[:just_signed_up] %>
  <p class="welcome">Welcome to our site!</p>
<% end %>
```

Si desea que un valor de flash se transfiera a otra solicitud, utilice el método `keep`:

```ruby
class MainController < ApplicationController
  # Let's say this action corresponds to root_url, but you want
  # all requests here to be redirected to UsersController#index.
  # If an action sets the flash and redirects here, the values
  # would normally be lost when another redirect happens, but you
  # can use 'keep' to make it persist for another request.
  def index
    # Will persist all flash values.
    flash.keep
 
    # You can also use a key to keep only some kind of value.
    # flash.keep(:notice)
    redirect_to users_url
  end
end
```



