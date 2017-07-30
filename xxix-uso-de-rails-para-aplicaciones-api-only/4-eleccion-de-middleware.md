# 4- Elección de middleware

Una aplicación de API viene con el siguiente middleware de forma predeterminada:

* `Rack::Sendfile`
* `ActionDispatch::Static`
* `ActionDispatch::Executor`
* `ActiveSupport::Cache::Strategy::LocalCache::Middleware`
* `Rack::Runtime`
* `ActionDispatch::RequestId`
* `ActionDispatch::RemoteIp`
* `Rails::Rack::Logger`
* `ActionDispatch::ShowExceptions`
* `ActionDispatch::DebugExceptions`
* `ActionDispatch::Reloader`
* `ActionDispatch::Callbacks`
* `ActiveRecord::Migration::CheckPending`
* `Rack::Head`
* `Rack::ConditionalGet`
* `Rack::ETag`
* `MyApi::Application::Routes`

Consulte la sección middleware interno de la guía `Rack` para obtener más información sobre ellos.

Otros complementos, incluido Active Record, pueden agregar middleware adicional. En general, estos middleware son agnósticos con el tipo de aplicación que está construyendo y tienen sentido en una aplicación de Rails de sólo API.

Puede obtener una lista de todos los middleware en su aplicación a través de:

```ruby
$ rails middleware
```

### 4.1 Uso del middleware de caché

De forma predeterminada, Rails agregará un middleware que proporciona un almacén de caché basado en la configuración de su aplicación \(`memcache` de forma predeterminada\). Esto significa que el caché HTTP incorporado dependerá de él.

Por ejemplo, utilizando el método `stale?` :

```ruby
def show
  @post = Post.find(params[:id])
 
  if stale?(last_modified: @post.updated_at)
    render json: @post
  end
end
```

La llamada a `stale?` comparará el encabezado `If-Modified-Since` en la solicitud con `@post.updated_at`. Si el encabezado es más reciente que el último modificado, esta acción devolverá una respuesta "304 no modificada". De lo contrario, mostrará la respuesta e incluirá un encabezado `Last-Modified `en él.

Normalmente, este mecanismo se utiliza en una base por cliente. El middleware de caché nos permite compartir este mecanismo de almacenamiento en caché entre los clientes. Podemos habilitar el almacenamiento en caché de varios clientes en la llamada para  `stale?`:

```ruby
def show
  @post = Post.find(params[:id])
 
  if stale?(last_modified: @post.updated_at, public: true)
    render json: @post
  end
end
```

Esto significa que el middleware de caché almacenará el valor `Last-Modified` para una URL en la caché de Rails y agregará un encabezado `If-Modified-Since` a cualquier solicitud de entrada posterior para la misma URL.

Piense en ello como el almacenamiento en caché de páginas mediante la semántica HTTP.

### 4.2 Uso de `Rack::Sendfile`

Cuando utiliza el método `send_file` dentro de un controlador de Rails, establece el encabezado `X-Sendfile`. `Rack::Sendfile` es responsable de enviar el archivo.

Si su servidor front-end admite el envío acelerado de archivos, `Rack::Sendfile` descargará el trabajo de envío de archivos real al servidor de aplicaciones para el usuario.

Puede configurar el nombre del encabezado que utiliza su servidor front-end con este fin utilizando `config.action_dispatch.x_sendfile_header` en el archivo de configuración del entorno apropiado.

Puede obtener más información sobre cómo utilizar `Rack::Sendfile` con front-ends populares en la [documentación](http://www.rubydoc.info/github/rack/rack/master/Rack/Sendfile) `Rack::Sendfile`.

Estos son algunos valores para esta cabecera para algunos servidores populares, una vez que estos servidores están configurados para admitir el envío acelerado de archivos:

```ruby
# Apache and lighttpd
config.action_dispatch.x_sendfile_header = "X-Sendfile"
 
# Nginx
config.action_dispatch.x_sendfile_header = "X-Accel-Redirect"
```

Asegúrese de configurar su servidor para que admita estas opciones siguiendo las instrucciones de la documentación de `Rack::Sendfile`.

### 4.3 Uso de `ActionDispatch::Request`

`ActionDispatch::Request#Params` tomará los parámetros del cliente en el formato JSON y los hará disponibles en su controlador dentro de los parámetros.

Para usar esto, su cliente necesitará hacer una petición con parámetros codificados por JSON y especificar el `Content-Type` como `application/json`.

Aquí hay un ejemplo en jQuery:

```js
jQuery.ajax({
  type: 'POST',
  url: '/people',
  dataType: 'json',
  contentType: 'application/json',
  data: JSON.stringify({ person: { firstName: "Yehuda", lastName: "Katz" } }),
  success: function(json) { }
});
```

`ActionDispatch::Request` verá el Content-Type y sus parámetros serán:

```ruby
{ :person => { :firstName => "Yehuda", :lastName => "Katz" } }
```

### 4.4 Otros middleware

Rails se envía con un número de otros middleware que puede que desee utilizar en una aplicación de la API, especialmente si uno de los clientes de su API es el navegador:

* `Rack::MethodOverride`

* `ActionDispatch::Cookies`
* `ActionDispatch::Flash`
* Para el manejo de sesiones:
  * `ActionDispatch::Session::CacheStore`
  * `ActionDispatch::Session::CookieStore`
  * `ActionDispatch::Session::MemCacheStore`

Cualquiera de estos middleware se puede agregar a través de:

```ruby
config.middleware.use Rack::MethodOverride
```

### 4.5 Eliminación del middleware

Si no desea utilizar un middleware que se incluye de forma predeterminada en el conjunto de middleware solo API, puede quitarlo con:

```ruby
config.middleware.delete ::Rack::Sendfile
```

Tenga en cuenta que la eliminación de estos middlewares eliminará la compatibilidad con ciertas funciones de Action Controller.











