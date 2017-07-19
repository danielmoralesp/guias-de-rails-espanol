# 6- Pruebas de integración

Las pruebas de integración se utilizan para probar cómo interactúan varias partes de su aplicación. Generalmente se utilizan para probar flujos de trabajo importantes dentro de nuestra aplicación.

Para crear las pruebas de integración de Rails, usamos el directorio de `test/integration` para nuestra aplicación. Rails proporciona un generador para crear un esqueleto de prueba de integración por nosotros.

```ruby
$ bin/rails generate integration_test user_flows
      exists  test/integration/
      create  test/integration/user_flows_test.rb
```

Esto es lo que muestra una prueba de integración recién generada:

```ruby
require 'test_helper'
 
class UserFlowsTest < ActionDispatch::IntegrationTest
  # test "the truth" do
  #   assert true
  # end
end
```

Aquí la prueba hereda de `ActionDispatch::IntegrationTest`. Esto hace que algunos helpers adicionales estén disponibles para que los utilicemos en nuestras pruebas de integración.

### 6.1 Helpers disponibles para pruebas de integración

Además de los helpers de pruebas estándar, heredando de `ActionDispatch::IntegrationTest` vendrá con algunos helpers adicionales disponibles al escribir las pruebas de integración. Vamos a presentar brevemente a las tres categorías de helpers que podemos elegir.

Para tratar con el corredor de pruebas de integración, vea `ActionDispatch::Integration::Runner`.

Al realizar las solicitudes, dispondremos de `ActionDispatch::Integration::RequestHelpers` para nuestro uso.

Si necesitamos modificar la sesión o el estado de nuestra prueba de integración, eche un vistazo a `ActionDispatch::Integration::Session` para ver ayudas.

### 6.2 Implementación de una prueba de integración

Añadamos una prueba de integración a nuestra aplicación de blog. Comenzaremos con un flujo de trabajo básico para crear un nuevo artículo de blog, para verificar que todo funciona correctamente.

Empezaremos generando nuestro esqueleto de integración:

```ruby
$ bin/rails generate integration_test blog_flow
```

Debería haber creado un marcador de archivo de prueba para nosotros. Con la salida del comando anterior deberíamos ver:

```ruby
invoke  test_unit
create    test/integration/blog_flow_test.rb
```

Ahora vamos a abrir ese archivo y escribir su primera aserción:

```ruby
require 'test_helper'
 
class BlogFlowTest < ActionDispatch::IntegrationTest
  test "can see the welcome page" do
    get "/"
    assert_select "h1", "Welcome#index"
  end
end
```

Echaremos un vistazo a `assert_select` para consultar el HTML resultante de una solicitud en la sección "Vistas de prueba" a continuación. Se utiliza para probar la respuesta de nuestra solicitud afirmando la presencia de elementos clave HTML y su contenido.

Cuando visitamos nuestro path de raíz, debemos ver `welcome/index.html.erb` renderizado por la vista. Así que esta aserción debe pasar.

#### 6.2.1 Creación de integración de artículos

Qué tal probar nuestra capacidad de crear un nuevo artículo en nuestro blog y ver el artículo resultante.

```ruby
test "can create an article" do
  get "/articles/new"
  assert_response :success
 
  post "/articles",
    params: { article: { title: "can create", body: "article successfully." } }
  assert_response :redirect
  follow_redirect!
  assert_response :success
  assert_select "p", "Title:\n  can create"
end
```

Vamos a romper esta prueba para poder entenderla.

Empezamos llamando a la acción `:new` en nuestro controlador de artículos. Esta respuesta debe tener éxito.

Después de esto hacemos una solicitud de post a la acción `:create` de nuestro controlador de artículos:

```ruby
post "/articles",
  params: { article: { title: "can create", body: "article successfully." } }
assert_response :redirect
follow_redirect!
```

Las dos líneas que siguen a la petición son para manejar la redirección que configuramos al crear un nuevo artículo.

> ¡No olvide llamar a `follow_redirect`! Si planea realizar solicitudes posteriores después de realizar una redirección.

Finalmente podemos afirmar\(assert\) que nuestra respuesta fue exitosa y nuestro nuevo artículo es legible en la página.

#### 6.2.2 Seguir avanzando

Hemos podido probar con éxito un flujo de trabajo muy pequeño para visitar nuestro blog y crear un nuevo artículo. Si quisiéramos llevar esto más lejos, podríamos añadir pruebas para comentar, eliminar artículos o editar comentarios. Las pruebas de integración son un gran lugar para experimentar con todo tipo de casos de uso para nuestras aplicaciones.





