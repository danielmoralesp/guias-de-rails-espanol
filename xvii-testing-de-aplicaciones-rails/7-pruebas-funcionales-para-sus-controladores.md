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

Las pruebas funcionales no verifican si el tipo de petición especificado es aceptado por la acción, estamos más preocupados aqui por el resultado. Las pruebas de solicitud existen para este caso de uso con el fin de hacer que sus pruebas sean más útiles.













