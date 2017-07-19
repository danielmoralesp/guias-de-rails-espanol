# 5- Pruebas del sistema

Las pruebas del sistema permiten probar las interacciones del usuario con su aplicación, ejecutando pruebas en un navegador real o por medio de los headers. Las pruebas del sistema utilizan `Capybara` como base.

Para crear las pruebas del sistema Rails, utilice el directorio `test/system` de su aplicación. Rails proporciona un generador para crear un esqueleto de prueba del sistema por usted.

```ruby
$ bin/rails generate system_test users
      invoke test_unit
      create test/system/users_test.rb
```

Esto es lo que parece una prueba de sistema recién generada:

```ruby
require "application_system_test_case"
 
class UsersTest < ApplicationSystemTestCase
  # test "visiting the index" do
  #   visit users_url
  #
  #   assert_selector "h1", text: "Users"
  # end
end
```

De forma predeterminada, las pruebas del sistema se ejecutan con el controlador Selenium, mediante el navegador Chrome y un tamaño de pantalla de 1400x1400. La siguiente sección explica cómo cambiar la configuración predeterminada.

### 5.1 Cambio de la configuración predeterminada

Rails hace que el cambio de la configuración predeterminada para las pruebas del sistema sea muy sencillo. Toda la configuración se abstrae para que pueda concentrarse en escribir sus pruebas.

Cuando genera una nueva aplicación o scaffold, se crea un archivo `application_system_test_case.rb` en el directorio de text. Aquí es donde debe vivir toda la configuración de las pruebas del sistema.

Si desea cambiar la configuración predeterminada, simplemente puede cambiar lo que las pruebas del sistema hacen como "driven by". Digamos que desea cambiar el controlador de Selenium a Poltergeist. Primero agrega la gema Poltergeist a tu Gemfile. A continuación, en su archivo `application_system_test_case.rb` haga lo siguiente:

```ruby
require "test_helper"
require "capybara/poltergeist"
 
class ApplicationSystemTestCase < ActionDispatch::SystemTestCase
  driven_by :poltergeist
end
```

El nombre del controlador es un argumento obligatorio para `driven_by`. Los argumentos opcionales que se pueden pasar a `driven_by` son `:using` para el navegador \(esto sólo será utilizado por Selenium\), `:screen_size` para cambiar el tamaño de la pantalla para capturas de pantalla, y `:options` que se pueden usar para establecer opciones soportadas por el driver.

```ruby
require "test_helper"
 
class ApplicationSystemTestCase < ActionDispatch::SystemTestCase
  driven_by :selenium, using: :firefox
end
```

Si su configuración de Capybara requiere más configuración que la proporcionada por Rails, esta configuración adicional podría agregarse al archivo `application_system_test_case.rb`.

Consulte la [documentación de Capybara](https://github.com/teamcapybara/capybara#setup) para obtener información adicional.

### 5.2 Helper de captura de pantalla

El ScreenshotHelper es un ayudante diseñado para capturar  screenchots de tus pruebas. Esto puede ser útil para ver el navegador en el punto en el que falló una prueba o para ver capturas de pantalla más tarde para la depuración.

Se proporcionan dos métodos: `take_screenshot` y `take_failed_screenshot`. `take_failed_screenshot` se incluye automáticamente en `after_teardown` dentro de Rails.

El método de ayuda de `take_screenshot` se puede incluir en cualquier lugar de las pruebas para tomar una captura de pantalla del navegador.

### 5.3 Implementación de una prueba del sistema

Ahora vamos a agregar una prueba del sistema a nuestra aplicación de blog. Demostramos que escribimos una prueba del sistema visitando la página de index y creando un nuevo artículo de blog.

Si utilizó el scaffold, un esqueleto de test del sistema se creó automáticamente por usted. Si no usó el scaffold, comience creando un esqueleto de prueba del sistema.

```ruby
$ bin/rails generate system_test articles
```

Debería haber creado un marcador de archivo de prueba por nosotros. Con la salida del comando anterior debería ver:

```ruby
invoke  test_unit
create    test/system/articles_test.rb
```

Ahora vamos a abrir ese archivo y escribir su primera aserción:

```ruby
require "application_system_test_case"
 
class ArticlesTest < ApplicationSystemTestCase
  test "viewing the index" do
    visit articles_path
    assert_selector "h1", text: "Articles"
  end
end
```

La prueba debe ver que hay un `h1` en el índice de artículos y pasa.

Ejecute las pruebas del sistema.

```ruby
bin/rails test:system
```

> De forma predeterminada, correr el comando `bin / rails test` no ejecutará las pruebas del sistema. Asegúrese de ejecutar `bin / rails test:system` para ejecutarlos realmente.

#### 5.3.1 Creación de articulos para testearlos como prueba de sistema

Ahora probemos el flujo para crear un nuevo artículo en nuestro blog.

```ruby
test "creating an article" do
  visit articles_path
 
  click_on "New Article"
 
  fill_in "Title", with: "Creating an Article"
  fill_in "Body", with: "Created this article successfully!"
 
  click_on "Create Article"
 
  assert_text "Creating an Article"
end
```

El primer paso es llamar a `articles_path`. Esto llevará la prueba a la página de índice de artículos.

Entonces click\_on "New Article" encontrará el botón "New Article" en la página index. Esto redireccionará el navegador a `/articles/new`.

A continuación, la prueba rellenará el título y el cuerpo del artículo con el texto especificado. Una vez rellenados los campos, se hace clic en "Crete Article" en el que se enviará una solicitud `POST` para crear el nuevo artículo en la base de datos.

Seremos redirigidos de nuevo a la página de índice de artículos y allí hacemos un assert para que el texto del título del artículo esté en la página de índice de artículos.

#### 5.3.2 Haciendo más

La belleza de las pruebas del sistema es que es similar a las pruebas de integración, ya que prueba la interacción del usuario con su controlador, modelo y vista, pero las pruebas del sistema son mucho más robustas y, de hecho, prueban su aplicación como si un usuario real la estuviera utilizando. A continuación, puede probar cualquier cosa que el propio usuario pueda hacer en su aplicación, como comentar, eliminar artículos, publicar artículos de borrador, etc.







