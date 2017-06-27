# 3- Rutas sin resources

Además del enrutamiento por resources, Rails tiene un poderoso apoyo para el enrutamiento de direcciones URL arbitrarias a las acciones. Aquí, no obtiene grupos de rutas generadas automáticamente por resources. En su lugar, configura cada ruta de su aplicación por separado.

Mientras que normalmente debe usar resources, todavía hay muchos lugares donde el enrutamiento simple es más apropiado. No hay necesidad de intentar cazar cada pieza de su aplicación en un marco de recursos si no es un buen ajuste.

En particular, el enrutamiento sencillo facilita el mapeo de las URL heredadas a las nuevas acciones de Rails.

### 3.1 Parámetros vinculados

Cuando configura una ruta regular, proporciona una serie de símbolos que Rails asigna a partes de una solicitud `HTTP` entrante. Por ejemplo, considere esta ruta:

```ruby
get 'photos(/:id)', to: :display
```

Si una solicitud entrante a `/photos/1` es procesada por esta ruta \(porque no ha coincidido con ninguna ruta anterior en el archivo\), entonces el resultado será invocar la acción de visualización del `PhotosController` y hacer que el parámetro final "`1`" este disponible como `params[:id]`. Esta ruta también encaminará la solicitud entrante de `/photos` a `PhotosController#display`, ya que el `:id` es un parámetro opcional, indicado por paréntesis.

### 3.2 Segmentos dinámicos

Puede configurar tantos segmentos dinámicos dentro de una ruta regular como desee. Cualquier segmento estará disponible para la acción como parte de `params`. Si configura esta ruta:

```ruby
get 'photos/:id/:user_id', to: 'photos#show'
```

Una ruta entrante de `/photos/1/2` será enviada a la acción `show` del `PhotosController`. `params[: id]` será "`1`", y `params[:user_id]` será "`2`".

> De forma predeterminada, los segmentos dinámicos no aceptan puntos, ya que el punto se utiliza como separador para las rutas formateadas. Si necesita usar un punto dentro de un segmento dinámico, añada una restricción que reemplace esto: por ejemplo, `id: /[^\/]+/` permite cualquier cosa menos una barra.

### 3.3 Segmentos estáticos

Puede especificar segmentos estáticos al crear una ruta, no añadiendo dos puntos a un fragmento:

```ruby
get 'photos/:id/with_user/:user_id', to: 'photos#show'
```

Esta ruta respondería a rutas como `/photos/1/with_user/2`. En este caso, `params` sería `{ controller: 'photos', action: 'show', id: '1', user_id: '2' }`.

### 3.4 Query string

Los parámetros también incluirán cualquier parámetro de la cadena de consulta. Por ejemplo, con esta ruta:

```ruby
get 'photos/:id', to: 'photos#show'
```

Una ruta entrante de `/photos/1?user_id=2` será enviada a la acción show del controlador `Photos`. `params` será `{ controller: 'photos', action: 'show', id: '1', user_id: '2' }`.

### 3.5 Definición de valores predeterminados

Puede definir valores predeterminados en una ruta suministrando un hash para la opción `:defaults`. Esto incluso se aplica a los parámetros que no especifica como segmentos dinámicos. Por ejemplo:

```ruby
get 'photos/:id', to: 'photos#show', defaults: { format: 'jpg' }
```

Rails coincidiría con las `photos/12` con la acción `show` de `PhotosController`, y establecería `params[:format]` a "`jpg`".

También puede utilizar valores predeterminados en un formato de bloque para definir los valores predeterminados para varios elementos:

```ruby
defaults format: :json do
  resources :photos
end
```

> No puede anular los valores predeterminados a través de parámetros de consulta, por razones de seguridad. Los únicos valores por defecto que se pueden sobreescribir son los segmentos dinámicos a través de la sustitución en la ruta de la URL.

### 3.6 Nombrar las rutas

Puede especificar un nombre para cualquier ruta utilizando la opción `:as`:

```ruby
get 'exit', to: 'sessions#destroy', as: :logout
```

Esto creará `logout_path` y `logout_url` como ayudantes nombrados en su aplicación. Llamando a `logout_path` retornará `/exit`

También puede utilizar esta opción para anular métodos de enrutamiento definidos por recursos, como estos:

```ruby
get ':username', to: 'users#show', as: :user
```

Esto definirá un método `user_path` que estará disponible en los controladores, ayudantes y vistas que irán a una ruta como `/bob`. Dentro de la acción `show` de `UsersController`, `params[:username]` contendrá el nombre de usuario para el usuario. cambie `:username` en la definición de ruta si no desea que su nombre de parámetro sea `:username`.

### 3.7 Limitaciones del verbo HTTP

En general, debe usar los métodos `get`, `post`, `put`, `patch` y `delete` para restringir una ruta a un verbo en particular. Puede utilizar el método de coincidencia con la opción `:via` para hacer coincidir varios verbos a la vez:

```ruby
match 'photos', to: 'photos#show', via: [:get, :post]
```

Puede hacer coincidir todos los verbos con una ruta particular usando `via: :all`

```ruby
match 'photos', to: 'photos#show', via: :all
```

> El enrutamiento de las solicitudes `GET` y `POST` a una sola acción tiene implicaciones de seguridad. En general, debe evitar enrutar todos los verbos a una acción a menos que tenga una buena razón para hacerlo.

> '`GET`' en Rails no comprobará el token CSRF. Nunca debe escribir en la base de datos desde solicitudes 'GET', para obtener más información, consulte la guía de seguridad sobre contramedidas CSRF.

### 3.8 Limitaciones de segmentos

Puede utilizar la opción `:constraints` para imponer un formato para un segmento dinámico:

```ruby
get 'photos/:id', to: 'photos#show', constraints: { id: /[A-Z]\d{5}/ }
```

Esta ruta coincidiría con rutas como `/photos/A12345,` pero no `/photos/893`. Puedes expresar más sucintamente la misma ruta de esta manera:

```ruby
get 'photos/:id', to: 'photos#show', id: /[A-Z]\d{5}/
```

`:constraints` toma expresiones regulares con la restricción de que los anchors con regexp no se pueden usar. Por ejemplo, la siguiente ruta no funcionará:

```ruby
get '/:id', to: 'articles#show', constraints: { id: /^\d/ }
```

Sin embargo, tenga en cuenta que no es necesario utilizar anchors porque todas las rutas están ancladas al principio.

Por ejemplo, las siguientes rutas permitirían que los artículos con valores `to_param` como `1-hello-world` que siempre empiecen con un número y los usuarios con valores `to_param` como david que nunca empiecen con un número para compartir el espacio de nombres raíz:

```ruby
get '/:id', to: 'articles#show', constraints: { id: /\d.+/ }
get '/:username', to: 'users#show'
```

### 3.9 Restricciones basadas en un request

También puede restringir una ruta basada en cualquier método en el objeto `Request` que devuelve una cadena.

```ruby
get 'photos', to: 'photos#index', constraints: { subdomain: 'admin' }
```

También puede especificar restricciones en un formulario de bloque:

```ruby
namespace :admin do
  constraints subdomain: 'admin' do
    resources :photos
  end
end
```

> Las restricciones de solicitud funcionan llamando a un método en el objeto `Request` con el mismo nombre que la clave hash y luego compare el valor devuelto con el valor hash. Por lo tanto, los valores de restricción deben coincidir con el tipo de retorno del método de objeto `Request` correspondiente. Por ejemplo: `constrains: {subdomain: 'api'}` coincidirá con un subdominio api como se esperaba, sin embargo, usando una restricción de símbolos: `{subdomain: :api}` no, porque `request.subdomain` devuelve '`api`' como String.

> Hay una excepción para la restricción de formato: mientras que es un método en el objeto `Request`, también es un parámetro opcional implícito en cada ruta. Las restricciones de segmento tienen prioridad y la restricción de formato sólo se aplica como tal cuando se aplica a través de un hash. Por ejemplo, `get 'foo'`, `constraints: {format: 'json'}` coincidirá con GET `/foo` porque el formato es opcional por defecto. Sin embargo, puedes usar un lambda como en `get 'foo'`, las restricciones: `lambda {| req | Req.format ==: json}` y la ruta sólo coincidirá con las solicitudes JSON explícitas.

### 3.10 Restricciones avanzadas

Si tiene una restricción más avanzada, puede proporcionar un objeto que responda a `matches?` que Rails debe usar. Digamos que querías enrutar a todos los usuarios de una lista negra al `BlacklistController`. Podrías hacerlo:

```ruby
class BlacklistConstraint
  def initialize
    @ips = Blacklist.retrieve_ips
  end
 
  def matches?(request)
    @ips.include?(request.remote_ip)
  end
end
 
Rails.application.routes.draw do
  get '*path', to: 'blacklist#index',
    constraints: BlacklistConstraint.new
end
```

También puede especificar restricciones como un lambda:

```ruby
Rails.application.routes.draw do
  get '*path', to: 'blacklist#index',
    constraints: lambda { |request| Blacklist.retrieve_ips.include?(request.remote_ip) }
end
```

Ambos metodos `matches?` y el lambda obtiene el objeto request como argumento.

### 3.11 Rutas globales y segmentos comodín \(wildcards\)

La globalización de las rutas es una forma de especificar que un parámetro en particular debe coincidir con todas las partes restantes de una ruta. Por ejemplo:

```ruby
get 'photos/*other', to: 'photos#unknown'
```

Esta ruta coincidiría con las `photos/12` o `/photos/long/path/to/12`, configurando` params[:other]` a "`12`" o "`long/path/to/12`". Los fragmentos prefijados con una estrella se llaman "segmentos comodín".

Los segmentos comodín pueden ocurrir en cualquier lugar de una ruta. Por ejemplo:

```ruby
get 'books/*section/:title', to: 'books#show'
```

Coincidiría con `books/some/section/last-words-a-memoir` con `params[:section]` es igual a '`some/section`', y `params[:title]` igual a '`last-words-a-memoir`'.

Técnicamente, una ruta puede tener incluso más de un segmento comodín. El asignador asigna segmentos a los parámetros de una manera intuitiva. Por ejemplo:

```ruby
get '*a/foo/*b', to: 'test#index'
```

Sería igual a `zoo/woo/foo/bar/baz` con `params[:a]` igual a '`zoo/woo`', y `params[:b]` es igual a `'bar/baz`'.

Al solicitar '`/foo/bar.json'`, sus `params[:pages]` serán iguales a `'foo/bar`' con el formato de solicitud de `JSON`. Si desea que el antiguo comportamiento 3.0.x, puede proporcionar el formato `:false` como este:

```ruby
get '*pages', to: 'pages#show', format: false
```

Si desea que el segmento de formato sea obligatorio, por lo que no se puede omitir, puede proporcionar formato` :true` como este:

```ruby
get '*pages', to: 'pages#show', format: true
```

### 3.12 Redireccionamiento

Puede redirigir cualquier ruta a otra ruta utilizando el ayudante de redirección en su enrutador:

```ruby
get '/stories', to: redirect('/articles')
```

También puede reutilizar segmentos dinámicos de la coincidencia en la ruta a redireccionar a:

```ruby
get '/stories/:name', to: redirect('/articles/%{name}')
```

También puede proporcionar un bloque para redirigir, que recibe los parámetros de ruta simbolizados y el objeto de la petición:

```ruby
get '/stories/:name', to: redirect { |path_params, req| "/articles/#{path_params[:name].pluralize}" }
get '/stories', to: redirect { |path_params, req| "/articles/#{req.subdomain}" }
```

Tenga en cuenta que la redirección predeterminada es un redirect 301 "Moved Permanently". Tenga en cuenta que algunos navegadores web o servidores proxy almacenarán en caché este tipo de redireccionamiento, haciendo que la página antigua sea inaccesible. Puede usar la opción `:status` para cambiar el estado de la respuesta:

```ruby
get '/stories/:name', to: redirect('/articles/%{name}', status: 302)
```

En todos estos casos, si no proporciona el host principal \(`http://www.example.com`\), Rails tomará esos detalles de la solicitud actual.

### 3.13 Enrutamiento a aplicaciones Rack

En lugar de una cadena como '`articles#index`', que corresponde a la acción de índice en el `ArticlesController`, puede especificar cualquier aplicación Rack como punto final para un matcher:

```ruby
match '/application.js', to: MyRackApp, via: :all
```

Mientras MyRackApp responda a la llamada y devuelva un \[status, headers, body\], el enrutador no sabrá la diferencia entre la aplicación Rack y una acción. Este es un uso apropiado de `via: :all`, ya que usted querrá permitir que su aplicación Rack maneje todos los verbos que considere apropiados.

> Para los curiosos, '`articles#index`' se expande a `ArticlesController.action(:index)`, que devuelve una aplicación de Rack válida.

Si especifica una aplicación Rack como punto final para un matcher, recuerde que la ruta no cambiará en la aplicación receptora. Con la siguiente ruta, la aplicación de Rack debe esperar que la ruta sea '`/admin`':

```ruby
match '/admin', to: AdminApp, via: :all
```

Si prefiere que su aplicación Rack reciba peticiones en la ruta raíz, utilice `mount`:

```ruby
mount AdminApp, at: '/admin'
```

### 3.14 Uso de root

Puede especificar Rails qué debe enrutar en '`/`' con el método `root`:

```ruby
root to: 'pages#main'
root 'pages#main' # shortcut for the above
```

Debe poner la ruta raíz en la parte superior del archivo, ya que es la ruta más popular y debe coincidir primero.

> La ruta raíz solo encamina solicitudes GET a la acción.

También puede utilizar la ruta raíz dentro de espacios de nombres y los ámbitos. Por ejemplo:

```ruby
namespace :admin do
  root to: "admin#index"
end
 
root to: "home#index"
```

### 3.15 Rutas de caracteres Unicode

Puede especificar rutas de carácter unicode directamente. Por ejemplo:

```ruby
get 'こんにちは', to: 'welcome#index'
```



