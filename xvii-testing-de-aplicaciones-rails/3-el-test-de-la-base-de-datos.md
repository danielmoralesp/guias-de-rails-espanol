# 3- Base de datos del testing

Casi todas las aplicaciones de Rails interactúan fuertemente con una base de datos y, como resultado, sus pruebas también necesitarán una base de datos para interactuar. Para escribir pruebas eficientes, necesitará entender cómo configurar esta base de datos y rellenarla con datos de ejemplo.

Por defecto, cada aplicación de Rails tiene tres entornos: desarrollo, testing y producción. La base de datos para cada uno de ellos se configura en `config/database.yml`.

Una base de datos de pruebas dedicada le permite configurar e interactuar con datos de prueba de forma aislada. De esta manera sus pruebas pueden manipular datos de prueba con confianza, sin preocuparse por los datos en las bases de datos de desarrollo o producción.

### 3.1 Mantenimiento del esquema de la base de datos de prueba

Para ejecutar las pruebas, la base de datos de prueba necesitará tener la estructura actual. El asistente de prueba comprueba si la base de datos de prueba tiene alguna migración pendiente. Intentará cargar su `db/schema.rb` o `db/structure.sql` en la base de datos de prueba. Si las migraciones aún están pendientes, se generará un error. Por lo general, esto indica que su esquema no se ha migrado completamente. La ejecución de las migraciones en la base de datos de desarrollo \(`bin/rails db:migrate`\) actualizará el esquema.

Si hubo modificaciones en las migraciones existentes, es necesario reconstruir la base de datos de prueba. Esto se puede hacer ejecutando `bin/ rails db:test:prepare`.

### 3.2 El Low-Down en los Fixtures

Para tener buenos test, tendrá que pensar un poco en la configuración de los datos de prueba. En Rails, puede manejar esto definiendo y personalizando los fixtures. Puede encontrar la documentación completa en la documentación de la [API de Fixtures](http://api.rubyonrails.org/v5.1.2/classes/ActiveRecord/FixtureSet.html).

#### 3.2.1 ¿Qué son los fixtures?

Fixtures es una palabra ficticia para los datos de la muestra. Los fixtures permiten llenar la base de datos de pruebas con datos predefinidos antes de que se ejecuten las pruebas. Los fixtures son independientes de la base de datos y están escritos en YAML. Hay un archivo por modelo.

> Los fixtures no están diseñados para crear todos los objetos que sus pruebas necesitan, y se administran mejor cuando se utilizan sólo para datos predeterminados que se pueden aplicar al caso común.

Encontrará los fixtures bajo su directorio de `test/fixtures`. Al ejecutar `rails generate model` para crear un nuevo modelo, Rails creará automáticamente el stub de fixtures en este directorio.

#### 3.2.2 YAML

Los fixtures con formato YAML son una manera amigable para el usuario de describir sus datos de muestra. Estos tipos de fixtures tienen la extensión `.yml` \(como en `users.yml`\).

He aquí un ejemplo de los fixtures en YAML:

```ruby
# lo & behold! I am a YAML comment!
david:
  name: David Heinemeier Hansson
  birthday: 1979-10-15
  profession: Systems development
 
steve:
  name: Steve Ross Kellock
  birthday: 1974-09-27
  profession: guy with keyboard
```

A cada fixture se le asigna un nombre seguido por una lista identada de llave/valor separados por dos puntos. Los registros suelen estar separados por una línea en blanco. Puede colocar comentarios en un archivo de fixtures se hace usando el carácter `#` en la primera columna.

Si está trabajando con asociaciones, simplemente puede definir un nodo de referencia entre dos fixtures diferentes. He aquí un ejemplo con una asociación `belongs_to / has_many`:

```ruby
# In fixtures/categories.yml
about:
  name: About
 
# In fixtures/articles.yml
first:
  title: Welcome to Rails!
  body: Hello world!
  category: about
```

Observe la clave `category` del artículo `first` que se encuentra en `fixtures/articles.yml` y que tiene un valor  `about`. Esto le dice a Rails que cargue la categoría que se encuentra en `fixtures/categories.yml`.

> Para que las asociaciones se refieran entre sí por nombres, puede utilizar el nombre del fixture en lugar de especificar el atributo `:id` en los fixtures asociados. Rails asignará automáticamente una clave primaria para que sea consistente entre las ejecuciones. Para obtener más información sobre este comportamiento de la asociación, lea la documentación de la API de Fixtures.

#### 3.2.3 ERB'in It Up

ERB le permite incrustar código Ruby dentro de plantillas. El formato de fixtures YAML se pre-procesa con ERB cuando Rails carga los fixtures. Esto le permite utilizar Ruby para ayudarle a generar algunos datos de ejemplo. Por ejemplo, el código siguiente genera un millar de usuarios:

```ruby
<% 1000.times do |n| %>
user_<%= n %>:
  username: <%= "user#{n}" %>
  email: <%= "user#{n}@example.com" %>
<% end %>
```

#### 3.2.4 Fixtures en acción

Rails carga automáticamente todos los accesorios del directorio de `test/fixtures` por defecto. La carga implica tres pasos:

* Elimine todos los datos existentes de la tabla correspondiente al fixture
* Cargue los datos del fixture en la tabla
* Vaciar los datos del ficture en un método en caso de que desee acceder a él directamente

> Para eliminar los datos existentes de la base de datos, Rails intenta deshabilitar los disparadores de integridad referencial \(como claves externas y restricciones de comprobación\). Si está recibiendo errores de permisos molestos en las pruebas en ejecución, asegúrese de que el usuario de la base de datos tenga privilegios para desactivar estos disparadores en el entorno de prueba. \(En PostgreSQL, sólo los superusuarios pueden deshabilitar todos los disparadores.\) Más información sobre los permisos de PostgreSQL [aquí](http://blog.endpoint.com/2012/10/postgres-system-triggers-error.html)\).

#### 3.2.5 Los fixtures son objetos de Active Record

Los fixtures son instancias de Active Record. Como se menciona en el punto \# 3 anterior, puede acceder directamente al objeto porque está disponible automáticamente como un método cuyo ámbito es local es el caso de prueba. Por ejemplo:

```ruby
# this will return the User object for the fixture named david
users(:david)
 
# this will return the property for david called id
users(:david).id
 
# one can also access methods available on the User class
david = users(:david)
david.call(david.partner)
```

Para obtener varios fixtures a la vez, puede pasar una lista de nombres de los fixtures. Por ejemplo:

```ruby
# this will return an array containing the fixtures david and steve
users(:david, :steve)
```



