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

