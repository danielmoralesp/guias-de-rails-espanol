# 2- bin/rails

Dado que Rails 5.0+ tiene comandos rake incorporados en los ejecutables, `bin/rails` es el nuevo valor predeterminado para ejecutar comandos.

Puede obtener una lista de tareas de `bin/rails` disponibles, que a menudo dependen de su directorio actual, escribiendo `bin/rails --help`. Cada tarea tiene una descripción, y debería ayudarle a encontrar lo que necesita.

```
$ bin/rails --help
Usage: rails COMMAND [ARGS]

The most common rails commands are:
generate    Generate new code (short-cut alias: "g")
console     Start the Rails console (short-cut alias: "c")
server      Start the Rails server (short-cut alias: "s")
...

All commands can be run with -h (or --help) for more information.

In addition to those commands, there are:
about                               List versions of all Rails ...
assets:clean[keep]                  Remove old compiled assets
assets:clobber                      Remove compiled assets
assets:environment                  Load asset compile environment
assets:precompile                   Compile all the assets ...
...
db:fixtures:load                    Loads fixtures into the ...
db:migrate                          Migrate the database ...
db:migrate:status                   Display status of migrations
db:rollback                         Rolls the schema back to ...
db:schema:cache:clear               Clears a db/schema_cache.yml file
db:schema:cache:dump                Creates a db/schema_cache.yml file
db:schema:dump                      Creates a db/schema.rb file ...
db:schema:load                      Loads a schema.rb file ...
db:seed                             Loads the seed data ...
db:structure:dump                   Dumps the database structure ...
db:structure:load                   Recreates the databases ...
db:version                          Retrieves the current schema ...
...
restart                             Restart app by touching ...
tmp:create                          Creates tmp directories ...
```

> También puede utilizar `bin/rails -T` para obtener la lista de tareas.

### 2.1 about

`bin/rails about` proporciona información sobre los números de versión de Ruby, RubyGems, Rails, los subcomponentes Rails, la carpeta de la aplicación, el nombre del entorno actual de Rails, el adaptador de base de datos de la aplicación y la versión del esquema. Es útil cuando necesitas pedir ayuda, comprobar si un parche de seguridad puede afectarte, o cuando necesitas algunas estadísticas para una instalación de Rails existente.

```
$ bin/rails about
About your application's environment
Rails version             5.1.0
Ruby version              2.2.2 (x86_64-linux)
RubyGems version          2.4.6
Rack version              2.0.1
JavaScript Runtime        Node.js (V8)
Middleware:               Rack::Sendfile, ActionDispatch::Static, ActionDispatch::Executor, ActiveSupport::Cache::Strategy::LocalCache::Middleware, Rack::Runtime, Rack::MethodOverride, ActionDispatch::RequestId, ActionDispatch::RemoteIp, Sprockets::Rails::QuietAssets, Rails::Rack::Logger, ActionDispatch::ShowExceptions, WebConsole::Middleware, ActionDispatch::DebugExceptions, ActionDispatch::Reloader, ActionDispatch::Callbacks, ActiveRecord::Migration::CheckPending, ActionDispatch::Cookies, ActionDispatch::Session::CookieStore, ActionDispatch::Flash, Rack::Head, Rack::ConditionalGet, Rack::ETag
Application root          /home/foobar/commandsapp
Environment               development
Database adapter          sqlite3
Database schema version   20110805173523
```

### 2.2 assets

Puede precompilar los activos en `app/assets` utilizando los recursos `bin/rails assets:precompile` y eliminar los recursos compilados más antiguos utilizando los recursos `bin/rails assets:clean`. La tarea `assets:clean` permite la implementación de deploys que aún pueden estar enlazando a un assets antiguo mientras se están construyendo los nuevos assets.

Si desea borrar completamente los elementos `public/assets`, puede utilizar los recursos `bin/rails assets:clobber`.

### 2.3 db

Las tareas más comunes del espacio de nombres `db`: bin / rails son `migrate` y `create` , y vale la pena probar todas las tareas de bin / rails de migración \(`up, down, redo, reset`\).  `bin / rails db:version` es útil cuando se soluciona el problema, diciéndole la versión actual de la base de datos.

Puede encontrar más información acerca de las migraciones en la guía Migraciones.

### 2.4 notes

`bin/rails notes` buscará en tu código los comentarios que comiencen con FIXME, OPTIMIZE o TODO. La búsqueda se realiza en archivos con extensión `.builder, .rb, .rake, .yml, .yaml, .ruby, .css, .js y .erb` para las anotaciones predeterminadas y personalizadas.

```
$ bin/rails notes
(in /home/foobar/commandsapp)
app/controllers/admin/users_controller.rb:
  * [ 20] [TODO] any other way to do this?
  * [132] [FIXME] high priority for next deploy
 
app/models/school.rb:
  * [ 13] [OPTIMIZE] refactor this code to make it faster
  * [ 17] [FIXME]
```

Puede agregar soporte para nuevas extensiones de archivo mediante la opción `config.annotations.register_extensions`, que recibe una lista de las extensiones con su regex correspondiente para que coincida con ella.

```ruby
config.annotations.register_extensions("scss", "sass", "less") { |annotation| /\/\/\s*(#{annotation}):?\s*(.*)$/ }
```

Si está buscando una anotación específica, diga FIXME, puede utilizar `bin/rails notes:fixme`. Tenga en cuenta que debe ser en minúscula el nombre de la anotación.

```
$ bin/rails notes:fixme
(in /home/foobar/commandsapp)
app/controllers/admin/users_controller.rb:
  * [132] high priority for next deploy
 
app/models/school.rb:
  * [ 17]
```

También puede usar anotaciones personalizadas en su código y listarlas usando `bin/rails notes:custom` especificando la anotación usando una variable de entorno ANNOTATION.

```
$ bin/rails notes:custom ANNOTATION=BUG
(in /home/foobar/commandsapp)
app/models/article.rb:
  * [ 23] Have to fix this one before pushing!
```

> Cuando se utilizan anotaciones específicas y anotaciones personalizadas, el nombre de anotación \(FIXME, BUG etc\) no se muestra en las líneas de salida.

De forma predeterminada, las notas de rails se verán en los directorios `app, config, db, lib y test`. Si desea buscar en otros directorios, puede configurarlos mediante la opción `config.annotations.register_directories`.

```ruby
config.annotations.register_directories("spec", "vendor")
```

También puede proporcionarlos como una lista separada por comas en la variable de entorno `SOURCE_ANNOTATION_DIRECTORIES.`

```
$ export SOURCE_ANNOTATION_DIRECTORIES='spec,vendor'
$ bin/rails notes
(in /home/foobar/commandsapp)
app/models/user.rb:
  * [ 35] [FIXME] User should have a subscription at this point
spec/models/user_spec.rb:
  * [122] [TODO] Verify the user that has a subscription works
```

### 2.5 routes

Las rutas de rails mostrarán todas las rutas definidas, lo que es útil para rastrear problemas de enrutamiento en su aplicación o para darle una buena visión general de las URL de una aplicación con la que está tratando de familiarizarse.

### 2.6 test

> Una buena descripción de las pruebas unitarias en Rails se proporciona en la Guía para testing para aplicaciones de Rails

Rails viene con una suite de pruebas llamada Minitest. Rails debe su estabilidad al uso de testing. Las tareas disponibles en el espacio de nombres `test:` ayudan a ejecutar las diferentes pruebas que esperamos escribir.

### 2.7 tmp

El directorio `Rails.root/tmp` es, al igual que el directorio `* nix/tmp`, es el lugar de almacenamiento de los archivos temporales como archivos de `id` de proceso y acciones en caché.

Las tareas `tmp:`  le ayudarán a despejar y crear el directorio `Rails.root/tmp`:

* `rails tmp:cache:clear` borra tmp/cache.
* `rails tmp:sockets:clear` borra tmp/sockets.
* `rails tmp:clear` borra todos los archivos de caché y sockets.
* `rails tmp:create` crea los directorios tmp para el caché, sockets y pids.

### 2.8 Miscellaneous

* `rails stats` es ideal para ver las estadísticas de su código, mostrando cosas como KLOC \(miles de líneas de código\) y su código para probar la relación.
* `rails secret` le dará una clave pseudoaleatoria para usar para su secreto de sesión.
* `rails time:zones:all` lista todas las zonas horarias que Rails conoce.

### 2.9 Tareas de rake personalizadas

Las tareas de rake personalizadas tienen una extensión `.rake` y se colocan en `Rails.root/lib/tasks`. Puede crear estas tareas de rastreo personalizadas con el comando `bin/rails generate task`.

```ruby
desc "I am short, but comprehensive description for my cool task"
task task_name: [:prerequisite_task, :another_task_we_depend_on] do
  # All your magic here
  # Any valid Ruby code is allowed
end
```

Para pasar argumentos a su tarea de rack personalizada:

```ruby
task :task_name, [:arg_1] => [:prerequisite_1, :prerequisite_2] do |task, args|
  argument_1 = args.arg_1
end
```

Puede agrupar las tareas colocándolas en espacios de nombres:

```ruby
namespace :db do
  desc "This task does nothing"
  task :nothing do
    # Seriously, nothing
  end
end
```

La invocación de las tareas se verá así:

```
$ bin/rails task_name
$ bin/rails "task_name[value 1]" # entire argument string should be quoted
$ bin/rails db:nothing
```

> Si tiene la necesidad de interactuar con sus modelos, y realizar consultas de base de datos y así sucesivamente, su tarea debe depender de la tarea de entorno, que cargará su código de aplicación.





