# 3- La Línea de Comando Avanzada de Rails

El uso más avanzado de la línea de comandos se enfoca en encontrar opciones útiles \(incluso sorprendentes a veces\) en las utilidades, y adaptarlas a sus necesidades y flujo de trabajo específico. Aquí están algunos trucos bajo la manga de Rails.

### 3.1 Rails con bases de datos y SCM

Al crear una nueva aplicación de Rails, tiene la opción de especificar qué tipo de base de datos y qué tipo de sistema de administración de código fuente va a utilizar su aplicación. Esto le ahorrará unos minutos, y ciertamente muchas pulsaciones de tecla.

Vamos a ver lo que una opción `--git` y una opción `--database=postgresql` hará por nosotros:

```
$ mkdir gitapp
$ cd gitapp
$ git init
Initialized empty Git repository in .git/
$ rails new . --git --database=postgresql
      exists
      create  app/controllers
      create  app/helpers
...
...
      create  tmp/cache
      create  tmp/pids
      create  Rakefile
add 'Rakefile'
      create  README.md
add 'README.md'
      create  app/controllers/application_controller.rb
add 'app/controllers/application_controller.rb'
      create  app/helpers/application_helper.rb
...
      create  log/test.log
add 'log/test.log'
```

Tuvimos que crear el directorio gitapp e inicializar un repositorio git vacío antes de que Rails agregara los archivos creados a nuestro repositorio. Veamos lo que poner en nuestra configuración de base de datos:

```
$ cat config/database.yml
# PostgreSQL. Versions 9.1 and up are supported.
#
# Install the pg driver:
#   gem install pg
# On OS X with Homebrew:
#   gem install pg -- --with-pg-config=/usr/local/bin/pg_config
# On OS X with MacPorts:
#   gem install pg -- --with-pg-config=/opt/local/lib/postgresql84/bin/pg_config
# On Windows:
#   gem install pg
#       Choose the win32 build.
#       Install PostgreSQL and put its /bin directory on your path.
#
# Configure Using Gemfile
# gem 'pg'
#
development:
  adapter: postgresql
  encoding: unicode
  database: gitapp_development
  pool: 5
  username: gitapp
  password:
...
...
```

También generó algunas líneas en nuestra configuración de database.yml correspondiente a nuestra elección de PostgreSQL para la base de datos.

> La única captura con el uso de las opciones de SCM es que usted tiene que hacer primero el directorio de su aplicación, luego inicializar su SCM, entonces puede ejecutar el comando `rails new` para generar la base de su aplicación.



