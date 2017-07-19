# 2- Introducción a las pruebas

El soporte para hacer testing en Rails se realizo desde el principio. No fue como una epifania tipo "oh, vamos a hacer el soporte para las pruebas de ejecución porque son nuevos y cool"

### 2.1 Rails se prepara para los testing desde la palabra Go

Rails crea un directorio de testing para usted tan pronto como crea un proyecto de Rails utilizando los `rails new application_name`. Si enlista el contenido de este directorio, verá:

```ruby
$ ls -F test
controllers/           helpers/               
mailers/               system/                                       test_helper.rb
fixtures/              integration/           
models/                application_system_test_case.rb
```

Los helpers, los mailers y los directorios de los modelos están destinados a realizar pruebas para los ayudantes de la vista, los mailers y los modelos, respectivamente. El directorio de controladores está destinado a realizar pruebas para controladores, rutas y vistas. El directorio de integración está destinado a realizar pruebas de interacciones entre controladores.

El directorio de prueba del sistema contiene pruebas del sistema, que se utilizan para la prueba completa del navegador de su aplicación. Las pruebas del sistema le permiten probar su aplicación de la misma manera que sus usuarios la experimentan y también ayudarle a probar su JavaScript. Las pruebas del sistema heredan de Capybara y realizan pruebas de navegador para su aplicación.

Los fixtures son una manera de organizar los datos de la prueba; Residen en el directorio `fixtures`.

También se creará un directorio de `jobs` cuando se genere una prueba asociada.

El archivo `test_helper.rb` contiene la configuración predeterminada para las pruebas.

El archivo `application_system_test_case.rb` contiene la configuración predeterminada para las pruebas del sistema.

### 2.2 El entorno de test

Por defecto, cada aplicación de Rails tiene tres entornos: desarrollo, test y producción.

La configuración de cada entorno se puede modificar de forma similar. En este caso, podemos modificar nuestro entorno de prueba cambiando las opciones encontradas en `config/environments/test.rb`.

> Sus pruebas se ejecutan bajo `RAILS_ENV=test`.

### 2.3 Rails usa Minitest

Si te acuerdas, usamos el comando `rails generate model` en la guía Getting Started with Rails. Hemos creado nuestro primer modelo, y entre otras cosas creó stubs de prueba en el directorio de prueba:

```ruby
$ bin/rails generate model article title:string body:text
...
create  app/models/article.rb
create  test/models/article_test.rb
create  test/fixtures/articles.yml
...
```

El stub de test predeterminado se encuentra en `test/models/article_test.rb` tiene este aspecto:

```ruby
require 'test_helper'

class ArticleTest < ActiveSupport::TestCase
  # test "the truth" do
  #   assert true
  # end
end
```

Un examen línea por línea de este archivo le ayudará a orientarlo a las pruebas código con Rails y la terminología.

```ruby
require 'test_helper'
```

Al solicitar este archivo, `test_helper.rb` se carga en la configuración predeterminada para ejecutar nuestras pruebas. Vamos a incluir esto con todas las pruebas que escribimos, por lo que cualquier método añadido a este archivo están disponibles para todas nuestras pruebas.

```ruby
class ArticleTest < ActiveSupport::TestCase
```

La clase `ArticleTest` define un caso de prueba porque hereda de `ActiveSupport::TestCase`. Por lo tanto, `ArticleTest` tiene todos los métodos disponibles de `ActiveSupport::TestCase`. Más adelante en esta guía, veremos algunos de los métodos que nos brinda.

Cualquier método definido en una clase heredada de `Minitest::Test` \(que es la superclase de `ActiveSupport::TestCase`\) que comienza con `test_` \(sensible a mayúsculas y minúsculas\) se llama simplemente test. Por lo tanto, los métodos definidos como `test_password` y `test_valid_password` son nombres de prueba legales y se ejecutan automáticamente cuando se ejecuta el caso de prueba.

Rails también agrega un método de prueba que toma un nombre de test y un bloque. Genera una prueba normal de `Minitest::Unit` con nombres de métodos prefijados con `test_`. Así que no tienes que preocuparte por nombrar los métodos, y puedes escribir algo como:

```ruby
test "the truth" do
  assert true
end
```

Que es mas o menos como escribir esto:

```ruby
def test_the_truth
  assert true
end
```

Aunque todavía puede utilizar definiciones de métodos habituales, el uso de la macro de prueba permite un nombre de prueba más legible.

> El nombre del método se genera reemplazando espacios con subrayados. El resultado no necesita ser un identificador de Ruby válido, aunque el nombre puede contener caracteres de puntuación, etc. Eso es porque en Ruby técnicamente cualquier cadena puede ser un nombre de método. Esto puede requerir el uso de `define_method` y enviar llamadas para funcionar correctamente, pero formalmente hay poca restricción en el nombre.

A continuación, echemos un vistazo a nuestra primera assertion:

```ruby
assert true
```

Una aserción es una línea de código que evalúa un objeto \(o expresión\) para los resultados esperados. Por ejemplo, una aserción puede comprobar:

* ¿Es este valor = a este valor?
* ¿Este objeto es nil?
* ¿Esta línea de código arroja una excepción?
* ¿Es la contraseña del usuario mayor de 5 caracteres?

Cada prueba puede contener una o más aserciones, sin restricción en cuanto a cuántas aserciones se permiten. Sólo cuando todas las aserciones sean exitosas, la prueba pasará.

#### 2.3.1 Su primera prueba fallida

Para ver cómo se informa un error de testing, puede agregar una prueba de error al caso de prueba `article_test.rb`.

```ruby
test "should not save article without title" do
  article = Article.new
  assert_not article.save
end
```

Vamos a ejecutar esta prueba recién agregada \(donde 6 es el número de línea donde se define la prueba\).

```ruby
$ bin/rails test test/models/article_test.rb:6
Run options: --seed 44656

# Running:

F

Failure:
ArticleTest#test_should_not_save_article_without_title [/path/to/blog/test/models/article_test.rb:6]:
Expected true to be nil or false


bin/rails test test/models/article_test.rb:6



Finished in 0.023918s, 41.8090 runs/s, 41.8090 assertions/s.

1 runs, 1 assertions, 1 failures, 0 errors, 0 skips
```

En la salida, F denota un fallo. Puede ver la traza correspondiente que se muestra en `Failure` junto con el nombre de la prueba que falla. Las siguientes líneas contienen el seguimiento de la pila seguido de un mensaje que menciona el valor real y el valor esperado por la aserción. Los mensajes de aserción predeterminados proporcionan suficiente información para ayudar a identificar el error. Para que el mensaje de fallo de aserción sea más legible, cada aserción proporciona un parámetro de mensaje opcional, como se muestra aquí:

```ruby
test "should not save article without title" do
  article = Article.new
  assert_not article.save, "Saved the article without a title"
end
```

Al ejecutar esta prueba se muestra el mensaje de aserción más amigable:

```ruby
Failure:
ArticleTest#test_should_not_save_article_without_title [/path/to/blog/test/models/article_test.rb:6]:
Saved the article without a title
```

Ahora para pasar esta prueba podemos agregar una validación de nivel de modelo para el campo de título.

```ruby
class Article < ApplicationRecord
  validates :title, presence: true
end
```

Ahora la prueba debe pasar. Vamos a verificar la ejecución de la prueba de nuevo:

```ruby
$ bin/rails test test/models/article_test.rb:6
Run options: --seed 31252

# Running:

.

Finished in 0.027476s, 36.3952 runs/s, 36.3952 assertions/s.

1 runs, 1 assertions, 0 failures, 0 errors, 0 skips
```

Ahora, si te fijaste, primero escribimos una prueba que falla para una funcionalidad deseada, luego escribimos algún código que agrega la funcionalidad y finalmente nos aseguramos de que nuestra prueba pase. Este enfoque para el desarrollo de software se conoce como [Test-Driven Development \(TDD\)](http://wiki.c2.com/?TestDrivenDevelopment).

#### 2.3.2 Lo que parece un error

Para ver cómo se informa de un error, aquí hay una prueba que contiene un error:

```ruby
test "should report error" do
  # some_undefined_variable is not defined elsewhere in the test case
  some_undefined_variable
  assert true
end
```

Ahora puedes ver más información en la salida de la consola ejecutando las pruebas:

```ruby
$ bin/rails test test/models/article_test.rb
Run options: --seed 1808

# Running:

.E

Error:
ArticleTest#test_should_report_error:
NameError: undefined local variable or method 'some_undefined_variable' for #<ArticleTest:0x007fee3aa71798>
    test/models/article_test.rb:11:in 'block in <class:ArticleTest>'


bin/rails test test/models/article_test.rb:9



Finished in 0.040609s, 49.2500 runs/s, 24.6250 assertions/s.

2 runs, 1 assertions, 0 failures, 1 errors, 0 skips
```

Observe el 'E' en la salida. Denota una prueba con error.

> La ejecución de cada método de prueba se detiene en cuanto se produce un error o un fallo de aserción y el conjunto de pruebas continúa con el siguiente método. Todos los métodos de prueba se ejecutan en orden aleatorio. La opción `config.active_support.test_order` se puede utilizar para configurar el orden de las pruebas.

Cuando una prueba falla se le presenta el backtrace correspondiente. De forma predeterminada, Rails filtra el backtrace y sólo imprime líneas relevantes para su aplicación. Esto elimina el ruido del framework y ayuda a centrarse en su código. Sin embargo, hay situaciones en las que desea ver el backtrace completo. Simplemente establezca el argumento `-b` \(o `--backtrace`\) para habilitar este comportamiento:

```ruby
$ bin/rails test -b test/models/article_test.rb
```

Si queremos que pase esta prueba, podemos modificarla para usar `assert_raises` así:

```ruby
test "should report error" do
  # some_undefined_variable is not defined elsewhere in the test case
  assert_raises(NameError) do
    some_undefined_variable
  end
end
```

Esta prueba debería pasar.

### 2.4 Aserciones disponibles

A estas alturas ya has visto algunas de las aserciones que están disponibles. Las aserciones son las abejas obreras de las pruebas. Son los que realmente realizan los chequeos para asegurarse de que las cosas van como están planificadas.

He aquí un extracto de las aserciones que puede utilizar con [Minitest](https://github.com/seattlerb/minitest), la biblioteca de pruebas predeterminada utilizada por Rails. El parámetro `[msg]` es un mensaje de cadena opcional que puede especificar para que los mensajes de error de prueba sean más claros.

| Assertion | Purpose |
| :--- | :--- |
| `assert( test, [msg] )` | Se asegura que`test` es true. |
| `assert_not( test, [msg] )` | Ensures that`test` es false. |
| `assert_equal( expected, actual, [msg] )` | Se asegura que`expected == actual` es true. |
| `assert_not_equal( expected, actual, [msg] )` | Se asegura que`expected != actual` es true. |
| `assert_same( expected, actual, [msg] )` | Se asegura que`expected.equal?(actual)` es true. |
| `assert_not_same( expected, actual, [msg] )` | Se asegura que`expected.equal?(actual)` es false. |
| `assert_nil( obj, [msg] )` | Se asegura que`obj.nil?` es true. |
| `assert_not_nil( obj, [msg] )` | Se asegura que `obj.nil?` es false. |
| `assert_empty( obj, [msg] )` | Se asegura que `obj`is`empty?`. |
| `assert_not_empty( obj, [msg] )` | Se asegura que`obj` no es `empty?`. |
| `assert_match( regexp, string, [msg] )` | Se asegura que el match entre un string y una expresion regular es true. |
| `assert_no_match( regexp, string, [msg] )` | Se asegura que un string no hace match con una expresion regular. |
| `assert_includes( collection, obj, [msg] )` | Se asegura que`obj` esta en `collection`. |
| `assert_not_includes( collection, obj, [msg] )` | Se asegura que`obj`no esta en`collection`. |
| `assert_in_delta( expected, actual, [delta], [msg] )` | Se asegura que los numeros `expected`and`actual`are within`delta`of each other. |
| `assert_not_in_delta( expected, actual, [delta], [msg] )` | Se asegura que the numbers`expected`and`actual`are not within`delta`of each other. |
| `assert_throws( symbol, [msg] ) { block }` | Se asegura que el bloque dado lanza un simbolo |
| `assert_raises( exception1, exception2, ... ) { block }` | Se asegura que el bloque dado plantea una de las excepciones dadas. |
| `assert_instance_of( class, obj, [msg] )` | Se asegura que`obj`es una instancia de la `class`. |
| `assert_not_instance_of( class, obj, [msg] )` | Se asegura que`obj`no es una instancia de la`class`. |
| `assert_kind_of( class, obj, [msg] )` | Se asegura que`obj`es una instancia de la `class`o es decendente de la misma. |
| `assert_not_kind_of( class, obj, [msg] )` | Se asegura que`obj`no es una instancia de la`class`y no es decendente de la misma |
| `assert_respond_to( obj, symbol, [msg] )` | Se asegura que`obj`responde a un `symbol`. |
| `assert_not_respond_to( obj, symbol, [msg] )` | Se asegura que`obj`no responde a un `symbol`. |
| `assert_operator( obj1, operator, [obj2], [msg] )` | Se asegura que`obj1.operator(obj2)` es true. |
| `assert_not_operator( obj1, operator, [obj2], [msg] )` | Se asegura que`obj1.operator(obj2)` es false. |
| `assert_predicate ( obj, predicate, [msg] )` | Se asegura que`obj.predicate` es true, por ejemplo.`assert_predicate str, :empty?` |
| `assert_not_predicate ( obj, predicate, [msg] )` | Se asegura que`obj.predicate` es false, por ejemplo.`assert_not_predicate str, :empty?` |
| `flunk( [msg] )` | Se asegura que sea failure. Esto es útil para marcar explícitamente una prueba que aún no ha terminado. |

Lo anterior es un subconjunto de afirmaciones que admite minitest. Para obtener una lista exhaustiva y actualizada, consulte la documentación de la [API de Minitest](http://docs.seattlerb.org/minitest/), específicamente [Minitest::Assertions](http://docs.seattlerb.org/minitest/Minitest/Assertions.html).

Debido a la naturaleza modular del framework de testing, es posible crear sus propias aserciones. De hecho, eso es exactamente lo que hace Rails. Incluye algunas aserciones especializadas para hacer su vida más fácil.

> Crear sus propias aserciones es un tema avanzado que no cubriremos en este tutorial.

### 2.5 Aserciones específicas de Rails

Rails añade algunas aserciones personalizadas propias al framework de minitest:

| Assertion | Purpose |
| :--- | :--- |
| [`assert_difference(expressions, difference = 1, message = nil) {...}`](http://api.rubyonrails.org/v5.1.2/classes/ActiveSupport/Testing/Assertions.html#method-i-assert_difference) | Probar la diferencia numérica entre el valor devuelto de una expresión como resultado de lo que se evalúa en el bloque cedido. |
| [`assert_no_difference(expressions, message = nil, &block)`](http://api.rubyonrails.org/v5.1.2/classes/ActiveSupport/Testing/Assertions.html#method-i-assert_no_difference) | Afirma que el resultado numérico de evaluar una expresión no se cambia antes y después de invocar el pasado en un bloque. |
| [`assert_nothing_raised { block }`](http://api.rubyonrails.org/v5.1.2/classes/ActiveSupport/TestCase.html#method-i-assert_nothing_raised) | Asegura que el bloque dado no plantea ninguna excepción. |
| [`assert_recognizes(expected_options, path, extras={}, message=nil)`](http://api.rubyonrails.org/v5.1.2/classes/ActionDispatch/Assertions/RoutingAssertions.html#method-i-assert_recognizes) | Afirma que el enrutamiento de la ruta de acceso dada se ha manejado correctamente y que las opciones analizadas \(dadas en el hash expected\_options\) coinciden con la ruta. Básicamente, afirma que Rails reconoce la ruta dada por expected\_options. |
| [`assert_generates(expected_path, options, defaults={}, extras = {}, message=nil)`](http://api.rubyonrails.org/v5.1.2/classes/ActionDispatch/Assertions/RoutingAssertions.html#method-i-assert_generates) | Afirma que las opciones proporcionadas pueden usarse para generar la ruta de acceso proporcionada. Este es el inverso de assert\_recognizes. El parámetro extras se utiliza para indicar a la solicitud los nombres y valores de los parámetros de petición adicionales que estarían en una cadena de consulta. El parámetro de mensaje le permite especificar un mensaje de error personalizado para los fallos de aserción. |
| [`assert_response(type, message = nil)`](http://api.rubyonrails.org/v5.1.2/classes/ActionDispatch/Assertions/ResponseAssertions.html#method-i-assert_response) | Afirma que la respuesta viene con un código de estado específico. Puede especificar :success para indicar 200-299, :redirect para indicar 300-399, :missing para indicar 404, o :errorpara coincidir con el rango 500-599. También puede pasar un número de estado explícito o su equivalente simbólico. Para obtener más información, consulte la lista completa de los códigos de estado y cómo. |
| [`assert_redirected_to(options = {}, message=nil)`](http://api.rubyonrails.org/v5.1.2/classes/ActionDispatch/Assertions/ResponseAssertions.html#method-i-assert_redirected_to) | Afirma que las opciones de redirección pasadas coinciden con las de la redirección que se llama en la acción más reciente. Este partido puede ser parcial, por ejemplo, que asert\_redirected\_to \(controller: "weblog"\) también coincide con la redirección de redirector\_to \(controlador: "weblog", acción: "show"\) y así sucesivamente. También puede pasar rutas nombradas como asassert\_redirected\_to root\_pathand Active Record objects such asassert\_redirected\_to @article. |











