# 1- Conceptos básicos de la línea de comandos

Hay algunos comandos que son absolutamente críticos para su uso diario de Rails. Enumerados el orden de cuánto probablemente los usarás son:

* `rails console`
* `rails server`
* `bin/rails`
* `rails generate`
* `rails dbconsole`
* `rails new app_name`

Todos los comandos pueden ejecutarse con `-h` o `--help `para obtener más información.

Vamos a crear una aplicación Rails simple para pasar a través de cada uno de estos comandos en el contexto.

### 1.1 rails new

Lo primero que queremos hacer es crear una nueva aplicación de Rails ejecutando el comando `rails new` después de instalar Rails.

> Usted puede instalar la gema de rails escribiendo `gem install rails`, si usted no la tiene ya.

```
$ rails new commandsapp
     create
     create  README.md
     create  Rakefile
     create  config.ru
     create  .gitignore
     create  Gemfile
     create  app
     ...
     create  tmp/cache
     ...
        run  bundle install
```

Rails te preparará con lo que parece una enorme cantidad de cosas para un comando tan pequeño! Ahora tiene toda la estructura de directorios de Rails con todo el código que necesita para ejecutar nuestra aplicación sencilla justo después de la caja.

### 1.2 rails server

El comando `rails server` lanza un servidor web llamado Puma que viene incluido con Rails. Lo usará cuando quiera acceder a su aplicación a través de un navegador web.

Sin más trabajo, el servidor de rails ejecutará nuestra nueva aplicación de Rails así:

    $ cd commandsapp
    $ bin/rails server
    => Booting Puma
    => Rails 5.1.0 application starting in development on http://0.0.0.0:3000
    => Run `rails server -h` for more startup options
    Puma starting in single mode...
    * Version 3.0.2 (ruby 2.3.0-p0), codename: Plethora of Penguin Pinatas
    * Min threads: 5, max threads: 5
    * Environment: development
    * Listening on tcp://localhost:3000
    Use Ctrl-C to stop

Con sólo tres órdenes que digitó ya tiene a un servidor Rails escuchando en el puerto 3000. Vaya a su navegador y abra `http://localhost:3000`, verá una aplicación básica de Rails en ejecución.

> También puede utilizar el alias "`s`" para iniciar el servidor: `rails s.`

El servidor se puede ejecutar en un puerto diferente mediante la opción `-p`. El entorno de desarrollo predeterminado se puede cambiar mediante `-e`.

```
$ bin/rails server -e production -p 4000
```

La opción` -b` enlaza Rails a la dirección IP especificada, por defecto es localhost. Puede ejecutar un servidor como un daemon pasando una opción `-d`.

### 1.3 rails generate

El comando `rails generate` usa plantillas para crear un montón de cosas. Correr `rails generate` genera por sí mismo una lista de generadores disponibles:

> También puede usar el alias "`g`" para invocar el comando del generador: `rails g`.

```
$ bin/rails generate
Usage: rails generate GENERATOR [args] [options]
 
...
...
 
Please choose a generator below.
 
Rails:
  assets
  controller
  generator
  ...
  ...
```

Usted puede instalar más generadores a través del generador de gemas, porciones de plugins que sin duda se instalará, e incluso puede crear su propio generador!

El uso de generadores le ahorrará una gran cantidad de tiempo mediante la escritura de código predeterminada, código que es necesario para que la aplicación funcione.

Hagamos nuestro propio controlador con el generador de controladores. Pero, ¿qué comando debemos usar? Vamos a preguntar al generador:

Todas las utilidades de la consola de Rails tienen texto de ayuda. Como con la mayoría de las utilidades de `*nix`, puede intentar agregar `--help` o `-h` al final, por ejemplo, `rails server --help`.$ bin/rails generate controller

    Usage: rails generate controller NAME [action action] [options]

    ...
    ...

    Description:
        ...

        To create a controller within a module, specify the controller name as a path like 'parent_module/controller_name'.

        ...

    Example:
        `rails generate controller CreditCards open debit credit close`

        Credit card controller with URLs like /credit_cards/debit.
            Controller: app/controllers/credit_cards_controller.rb
            Test:       test/controllers/credit_cards_controller_test.rb
            Views:      app/views/credit_cards/debit.html.erb [...]
            Helper:     app/helpers/credit_cards_helper.rb

El generador del controlador está esperando parámetros en la forma de `generate controller ControllerName action1 action2`. Hagamos un controlador `Greetings` con una acción de `hello`, que nos dirá algo agradable.

```
$ bin/rails generate controller Greetings hello
     create  app/controllers/greetings_controller.rb
      route  get "greetings/hello"
     invoke  erb
     create    app/views/greetings
     create    app/views/greetings/hello.html.erb
     invoke  test_unit
     create    test/controllers/greetings_controller_test.rb
     invoke  helper
     create    app/helpers/greetings_helper.rb
     invoke  assets
     invoke    coffee
     create      app/assets/javascripts/greetings.coffee
     invoke    scss
     create      app/assets/stylesheets/greetings.scss
```

¿Qué fue todo esto? Es un montón de directorios en nuestra aplicación, y creó un archivo controlador, un archivo de vista, un archivo de prueba funcional, un ayudante para la vista, un archivo de JavaScript y un archivo de hoja de estilo.

Compruebe el controlador y modifíquelo un poco \(en `app/controllers/greetings_controller.rb`\):

```ruby
class GreetingsController < ApplicationController
  def hello
    @message = "Hello, how are you today?"
  end
end
```

A continuación, la vista, para mostrar su mensaje \(en `app/views/greetings/hello.html.erb`\):

```html
<h1>A Greeting for You!</h1>
<p><%= @message %></p>
```

Encender el servidor utilizando el servidor de rails.

```
$ bin/rails server
=> Booting Puma...
```

La URL será `http://localhost:3000/greetings/hello.`

> Con una aplicación Rails normal, las URL seguirán generalmente el patrón `http://(host)/(controller)/(action)`, y una URL como http://\(host\)/\(controller\)  mostrará la acción index de ese controlador

Rails viene con un generador para los modelos de datos también.

```
$ bin/rails generate model
Usage:
  rails generate model NAME [field[:type][:index] field[:type][:index]] [options]
 
...
 
Active Record options:
      [--migration]            # Indicates when to generate migration
                               # Default: true
 
...
 
Description:
    Create rails files for model generator.
```

> Para obtener una lista de tipos de campos disponibles para el parámetro `type`, consulte la documentación del API para el método `add_column` del módulo `SchemaStatements`. El parámetro `index` genera un índice correspondiente para la columna.

Pero en lugar de generar un modelo directamente \(lo que haremos más adelante\), montaremos un scaffold. Un scaffold en Rails es un conjunto completo de modelo, migración de base de datos para ese modelo, controlador para manipularlo, vistas para ver y manipular los datos y un conjunto de pruebas para cada uno de los anteriores.

Vamos a crear un recurso simple llamado "HighScore" que mantendrá un registro de nuestra puntuación más alta en los videojuegos que jugamos.

```
$ bin/rails generate scaffold HighScore game:string score:integer
    invoke  active_record
    create    db/migrate/20130717151933_create_high_scores.rb
    create    app/models/high_score.rb
    invoke    test_unit
    create      test/models/high_score_test.rb
    create      test/fixtures/high_scores.yml
    invoke  resource_route
     route    resources :high_scores
    invoke  scaffold_controller
    create    app/controllers/high_scores_controller.rb
    invoke    erb
    create      app/views/high_scores
    create      app/views/high_scores/index.html.erb
    create      app/views/high_scores/edit.html.erb
    create      app/views/high_scores/show.html.erb
    create      app/views/high_scores/new.html.erb
    create      app/views/high_scores/_form.html.erb
    invoke    test_unit
    create      test/controllers/high_scores_controller_test.rb
    invoke    helper
    create      app/helpers/high_scores_helper.rb
    invoke    jbuilder
    create      app/views/high_scores/index.json.jbuilder
    create      app/views/high_scores/show.json.jbuilder
    invoke  assets
    invoke    coffee
    create      app/assets/javascripts/high_scores.coffee
    invoke    scss
    create      app/assets/stylesheets/high_scores.scss
    invoke  scss
   identical    app/assets/stylesheets/scaffolds.scss
```

El generador comprueba que existen los directorios para modelos, controladores, ayudantes, layouts, pruebas funcionales y de unidad, hojas de estilo, crea las vistas, el controlador, el modelo y la migración de bases de datos para HighScore \(creando la tabla y los campos `high_scores`\) Para el recurso, y nuevas pruebas para todo.

La migración requiere que migremos, es decir, ejecutar algún código Ruby \(viviendo en aqui `20130717151933_create_high_scores.rb`\) para modificar el esquema de nuestra base de datos. ¿Qué base de datos? La base de datos SQLite3 que Rails creará para usted cuando ejecute el comando `bin/rails db:migrate`. Hablaremos más acerca de `bin/rails` en profundidad más abajo.

```
$ bin/rails db:migrate
==  CreateHighScores: migrating ===============================================
-- create_table(:high_scores)
   -> 0.0017s
==  CreateHighScores: migrated (0.0019s) ======================================
```

> Hablemos de las pruebas unitarias. Las pruebas unitarias son código que prueba y hace afirmaciones sobre el código. En las pruebas unitarias, tomamos una pequeña parte del código, digamos un método de un modelo, y probamos sus entradas y salidas. Las pruebas unitarias son tus amigas. Cuanto antes se haga la paz con el hecho de que su calidad de vida aumentará drásticamente cuando pruebe unitariamente su código, mejor. Seriamente. Por favor visite la guía de pruebas para una mirada en profundidad a las pruebas unitarias.

Veamos la interfaz Rails creada para nosotros.

```ruby
$ bin/rails server
```

Vaya a su navegador y abra `http://localhost:3000/high_scores`, ahora podemos crear nuevos puntajes altos \(55.160 en Space Invaders!\)

### 1.4 rails console

El comando de consola le permite interactuar con su aplicación Rails desde la línea de comandos. En la parte inferior, la consola de rails usa IRB, así que si alguna vez lo usaste, estarás como en casa. Esto es útil para probar ideas rápidas con código y cambiar los datos del lado del servidor sin tocar el sitio web.

> También puede usar el alias "`c`" para invocar la consola: `rails c`.

Puede especificar el entorno en el que debe funcionar el comando de consola.

```
$ bin/rails console staging
```

Si desea probar algún código sin cambiar ningún dato, puede hacerlo invocando `rails console --sandobx`

```
$ bin/rails console --sandbox
Loading development environment in sandbox (Rails 5.1.0)
Any modifications you make will be rolled back on exit
irb(main):001:0>
```

#### 1.4.1 Los objetos app y helper

Dentro de la consola de rails tiene acceso a la aplicación y las instancias de ayuda.

Con el método `app` puede acceder a url y los ayudantes de ruta, así como hacer las solicitudes.

```
>> app.root_path
=> "/"
 
>> app.get _
Started GET "/" for 127.0.0.1 at 2014-06-19 10:41:57 -0300
...
```

Con el método auxiliar es posible acceder a Rails y a los ayudantes de su aplicación

```
>> helper.time_ago_in_words 30.days.ago
=> "about 1 month"
 
>> helper.my_custom_helper
=> "my custom helper"
```

### 1.5 rails dbconsole

`rails dbconsole` calcula la base de datos que utilizas y te deja en cualquier interfaz de línea de comandos que uses con ella \(y calcula los parámetros de la línea de comandos para darle también!\). Soporta MySQL \(incluyendo MariaDB\), PostgreSQL y SQLite3.

También puede utilizar el alias "`db`" para invocar la dbconsole: `rails db`.





