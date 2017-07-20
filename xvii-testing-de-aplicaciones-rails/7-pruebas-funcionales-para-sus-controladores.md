# 7- Pruebas funcionales para sus controladores

En Rails, probar las diversas acciones de un controlador es una forma de escribir pruebas funcionales. Recuerde que sus controladores manejan las solicitudes web entrantes en su aplicación y finalmente responden con una vista renderizada. Al escribir pruebas funcionales, está probando cómo sus acciones manejan las solicitudes y el resultado esperado o respuesta, que en algunos casos es una vista HTML.

### 7.1 Qué incluir en sus pruebas funcionales

Debe probar cosas como:

* ¿Fue la solicitud web exitosa?
* ¿Fue redirigido el usuario a la página correcta?
* ¿El usuario ha sido autenticado correctamente?
* ¿Fue almacenado el objeto correcto en la plantilla de respuesta?
* ¿Fue apropiado el mensaje mostrado al usuario en la vista?

La forma más fácil de ver las pruebas funcionales en acción es generar un controlador utilizando el generador scaffold:

```ruby
$ bin/rails generate scaffold_controller article title:string body:text
...
create  app/controllers/articles_controller.rb
...
invoke  test_unit
create    test/controllers/articles_controller_test.rb
...
```

Esto generará el código del controlador y las pruebas de un recurso\(resource\) de artículos. Puede echar un vistazo al archivo `articles_controller_test.rb` en el directorio `test/controllers`.

Si ya tiene un controlador y sólo desea generar el código scaffold de prueba para cada una de las siete acciones predeterminadas, puede utilizar el siguiente comando:

```ruby
$ bin/rails generate test_unit:scaffold article
...
invoke  test_unit
create test/controllers/articles_controller_test.rb
...
```

Echemos un vistazo a una prueba, `test_should_get_index` del archivo `articles_controller_test.rb`.

```ruby
# articles_controller_test.rb
class ArticlesControllerTest < ActionDispatch::IntegrationTest
  test "should get index" do
    get articles_url
    assert_response :success
  end
end
```

En la prueba `test_should_get_index`, Rails simula una solicitud en la acción llamada index, asegurándose de que la solicitud se realizó correctamente y también asegurándose de que se ha generado el cuerpo de respuesta correcto.

El método get inicia la solicitud web y rellena los resultados en `@response`. Puede aceptar hasta 6 argumentos:

La URI de la acción del controlador que está solicitando. Esto puede tener la forma de una cadena o un helper de ruta \(por ejemplo, `articles_url`\).

* La opcion `params:` con un hash de parámetros de petición para pasar a la acción \(por ejemplo, parámetros de cadena de consulta o variables de artículo\).
* `headers:` para establecer los encabezados que se pasarán con la solicitud.
* `env:` para personalizar el entorno de solicitud según sea necesario.
* `xhr:` si la solicitud es Ajax o no. Se puede establecer en true para marcar la solicitud como Ajax.
* `as:` para codificar la solicitud con diferentes tipos de contenido. Soporta `:json` por defecto.

Todos estos argumentos de palabra clave son opcionales.

Ejemplo: Llamando la acción :`show` y pasando un id de 12 como `params` y configurando el encabezado `HTTP_REFERER`:

```ruby
get article_url, params: { id: 12 }, headers: { "HTTP_REFERER" => "http://example.com/home" }
```

Otro ejemplo: Llamar a la acción `:update` , pasando un id de 12 como `params` como una solicitud de Ajax.

```ruby
patch article_url, params: { id: 12 }, xhr: true
```

> Si intenta ejecutar la prueba `test_should_create_article` desde `articles_controller_test.rb` fallará teniendo en cuenta que de la validación a nivel de modelo ha sido recién agregada y con razón.

Modificemos la prueba `test_should_create_article` en `articles_controller_test.rb` para que todos nuestros test pasen:

```ruby
test "should create article" do
  assert_difference('Article.count') do
    post articles_url, params: { article: { body: 'Rails is awesome!', title: 'Hello Rails' } }
  end

  assert_redirected_to article_path(Article.last)
end
```

Ahora puedes intentar ejecutar todas las pruebas y deben pasar.

Si siguió los pasos de la sección de autenticación básica, tendrá que agregar lo siguiente al bloque de instalación para hacer que todas las pruebas pasen:

```ruby
request.headers['Authorization'] = ActionController::HttpAuthentication::Basic.
  encode_credentials('dhh', 'secret')
```

### 7.2 Tipos de solicitud disponibles para pruebas funcionales

Si está familiarizado con el protocolo HTTP, sabrá que get es un tipo de solicitud. Hay 6 tipos de solicitud soportados en las pruebas funcionales de Rails:

* `get`
* `post`
* `patch`
* `put`
* `head`
* `delete`

Todos los tipos de solicitud tienen métodos equivalentes que puede utilizar. En una típica aplicación C.R.U.D.  va a utilizar `get`, `post`, `put` y `delete` más a menudo.

> Las pruebas funcionales no verifican si el tipo de petición especificado es aceptado por la acción, estamos más preocupados aqui por el resultado. Las pruebas de solicitud existen para este caso de uso con el fin de hacer que sus pruebas sean más útiles.

### 7.3 Prueba de solicitudes XHR \(AJAX\)

Para probar las solicitudes de AJAX, puede especificar la opción `xhr: true` para los metodos `get`, `post`, `put` y `delete`. Por ejemplo:

```ruby
test "ajax request" do
  article = articles(:one)
  get article_url(article), xhr: true
 
  assert_equal 'hello world', @response.body
  assert_equal "text/javascript", @response.content_type
end
```

### 7.4 Los Tres Hashes del Apocalipsis

Después de que se haya realizado y procesado una solicitud, tendrá 3 objetos Hash listos para su uso:

* cookies - Cualquier cookie que se establezca
* flash - Cualquier objeto que vive en el flash
* session  - Cualquier objeto que vive en variables de sesión

Como ocurre con los objetos Hash normales, puede acceder a los valores haciendo referencia a las claves por cadena. También puede hacer referencia a ellos por nombre de símbolo. Por ejemplo:

```ruby
flash["gordon"]               flash[:gordon]
session["shmession"]          session[:shmession]
cookies["are_good_for_u"]     cookies[:are_good_for_u]
```

### 7.5 Variables de instancia disponibles

También tiene acceso a tres variables de instancia en sus pruebas funcionales, después de hacer una solicitud:

* `@controller` - El controlador que procesa la solicitud
* `@request` - El objeto request
* `@response` - El objeto de respuesta

```ruby
class ArticlesControllerTest < ActionDispatch::IntegrationTest
  test "should get index" do
    get articles_url
 
    assert_equal "index", @controller.action_name
    assert_equal "application/x-www-form-urlencoded", @request.media_type
    assert_match "Articles", @response.body
  end
end
```

### 7.6 Configuración de encabezados y variables CGI

Los encabezados HTTP y las variables CGI se pueden pasar como encabezados:

```ruby
# setting an HTTP Header
get articles_url, headers: { "Content-Type": "text/plain" } # simulate the request with custom header
 
# setting a CGI variable
get articles_url, headers: { "HTTP_REFERER": "http://example.com/home" } # simulate the request with custom env variable
```

### 7.7 Testing a los avisos de flash

Si recuerdas uno de los puntos anteriores, uno de los Tres Hashes del Apocalipsis fue `flash`.

Queremos agregar un mensaje `flash` a nuestra aplicación de blog cada vez que alguien crea con éxito un nuevo artículo.

Comencemos añadiendo este assert a nuestra prueba `test_should_create_article`:

```ruby
test "should create article" do
  assert_difference('Article.count') do
    post article_url, params: { article: { title: 'Some title' } }
  end
 
  assert_redirected_to article_path(Article.last)
  assert_equal 'Article was successfully created.', flash[:notice]
end
```

Si ejecutamos nuestra prueba ahora, deberíamos ver un fallo:

```ruby
$ bin/rails test test/controllers/articles_controller_test.rb -n test_should_create_article
Run options: -n test_should_create_article --seed 32266
 
# Running:
 
F
 
Finished in 0.114870s, 8.7055 runs/s, 34.8220 assertions/s.
 
  1) Failure:
ArticlesControllerTest#test_should_create_article [/test/controllers/articles_controller_test.rb:16]:
--- expected
+++ actual
@@ -1 +1 @@
-"Article was successfully created."
+nil
 
1 runs, 4 assertions, 1 failures, 0 errors, 0 skips
```

Vamos a implementar el mensaje `flash` ahora en nuestro controlador. Nuestra acción `:create`  ahora debe verse así:

```ruby
def create
  @article = Article.new(article_params)
 
  if @article.save
    flash[:notice] = 'Article was successfully created.'
    redirect_to @article
  else
    render 'new'
  end
end
```

Ahora si ejecutamos nuestras pruebas, deberíamos verlo pasar:

```ruby
$ bin/rails test test/controllers/articles_controller_test.rb -n test_should_create_article
Run options: -n test_should_create_article --seed 18981
 
# Running:
 
.
 
Finished in 0.081972s, 12.1993 runs/s, 48.7972 assertions/s.
 
1 runs, 4 assertions, 0 failures, 0 errors, 0 skips
```

### 7.8 Juntándolo

En este punto, nuestro controlador de artículos testea las acciones `:index`, así como `:new` y `:create` . ¿Qué pasa con los datos existentes?

Escribamos una prueba para la acción `show`:

```ruby
test "should show article" do
  article = articles(:one)
  get article_url(article)
  assert_response :success
end
```

Recuerde nuestra discusión anterior en los fixtures, donde el método de los `articles()` nos dará acceso a nuestros fixtures de los articles.

¿Y qué hay sobre eliminar un artículo existente?

```ruby
test "should destroy article" do
  article = articles(:one)
  assert_difference('Article.count', -1) do
    delete article_url(article)
  end
 
  assert_redirected_to articles_path
end
```

También podemos añadir una prueba para actualizar un artículo existente.

```ruby
test "should update article" do
  article = articles(:one)
 
  patch article_url(article), params: { article: { title: "updated" } }
 
  assert_redirected_to article_path(article)
  # Reload association to fetch updated data and assert that title is updated.
  article.reload
  assert_equal "updated", article.title
end
```

Tenga en cuenta que estamos empezando a ver alguna duplicación en estas tres pruebas, ambos tienen acceso a los mismos datos del fixture del artículo. Podemos usar la filosofía D.R.Y. Esto mediante el uso de los métodos de configuración y desmontaje proporcionados por `ActiveSupport::Callbacks`.

Nuestra prueba ahora debe ser algo como lo que sigue. Ignore las otras pruebas por ahora, las dejamos por brevedad.

```ruby
require 'test_helper'
 
class ArticlesControllerTest < ActionDispatch::IntegrationTest
  # called before every single test
  setup do
    @article = articles(:one)
  end
 
  # called after every single test
  teardown do
    # when controller is using cache it may be a good idea to reset it afterwards
    Rails.cache.clear
  end
 
  test "should show article" do
    # Reuse the @article instance variable from setup
    get article_url(@article)
    assert_response :success
  end
 
  test "should destroy article" do
    assert_difference('Article.count', -1) do
      delete article_url(@article)
    end
 
    assert_redirected_to articles_path
  end
 
  test "should update article" do
    patch article_url(@article), params: { article: { title: "updated" } }
 
    assert_redirected_to article_path(@article)
    # Reload association to fetch updated data and assert that title is updated.
    @article.reload
    assert_equal "updated", @article.title
  end
end
```

Similar a otras devoluciones de llamada en Rails, los métodos de configuración y desmontaje también se pueden usar pasando un nombre de bloque, lambda o método como un símbolo para llamar.

### 7.9 Ayudantes de prueba

Para evitar la duplicación de código, puede agregar sus propios ayudantes de prueba. El ayudante sign puede ser un buen ejemplo:

```ruby
# test/test_helper.rb
 
module SignInHelper
  def sign_in_as(user)
    post sign_in_url(email: user.email, password: user.password)
  end
end
 
class ActionDispatch::IntegrationTest
  include SignInHelper
end
```

```ruby
require 'test_helper'
 
class ProfileControllerTest < ActionDispatch::IntegrationTest
 
  test "should show profile" do
    # helper is now reusable from any controller test case
    sign_in_as users(:david)
 
    get profile_url
    assert_response :success
  end
end
```













