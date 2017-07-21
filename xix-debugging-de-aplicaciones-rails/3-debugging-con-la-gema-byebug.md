# 3-  Debugging con la gema byebug

Cuando su código se comporta de manera inesperada, puede intentar imprimirlo en los logs o en la consola para diagnosticar el problema. Desafortunadamente, hay momentos en que este tipo de seguimiento de errores no es eficaz para encontrar la causa raíz de un problema. Cuando realmente necesita viajar en su código fuente en ejecución, el depurador es su mejor compañero.

El depurador también puede ayudarte si quieres aprender sobre el código fuente de Rails pero no sabes por dónde empezar. Sólo tiene que depurar cualquier solicitud de su aplicación y utilizar esta guía para aprender a moverse en el código que ha escrito en el código Rails subyacente.

### 3.1 Configuración

Puede usar la gema byebug para establecer puntos de interrupción \(breakpoints\) y pasar a través del código en vivo en Rails. Para instalarlo, ejecute:

```ruby
$ gem install byebug
```

Dentro de cualquier aplicación Rails, puede invocar el depurador llamando al método byebug.

He aquí un ejemplo:

```ruby
class PeopleController < ApplicationController
  def new
    byebug
    @person = Person.new
  end
end
```

### 3.2 El Shell

Tan pronto como su aplicación llame al método byebug, el debugger se iniciará en una shell de debbug dentro de la ventana de terminal donde lanzó su servidor de aplicaciones, y se colocará en el prompt del debugger \(byebug\). Antes de la solicitud, se mostrará el código alrededor de la línea que está a punto de ejecutarse y la línea actual se marcará con '`=>`', así:

```ruby
[1, 10] in /PathTo/project/app/controllers/articles_controller.rb
    3:
    4:   # GET /articles
    5:   # GET /articles.json
    6:   def index
    7:     byebug
=>  8:     @articles = Article.find_recent
    9:
   10:     respond_to do |format|
   11:       format.html # index.html.erb
   12:       format.json { render json: @articles }

(byebug)
```

Si llegaste allí por una solicitud del navegador, la pestaña del navegador que contiene la solicitud se congelará hasta que el depurador haya terminado y el trace haya terminado de procesar toda la solicitud.

Por ejemplo:

```ruby
=> Booting Puma
=> Rails 5.1.0 application starting in development on http://0.0.0.0:3000
=> Run `rails server -h` for more startup options
Puma starting in single mode...
* Version 3.4.0 (ruby 2.3.1-p112), codename: Owl Bowl Brawl
* Min threads: 5, max threads: 5
* Environment: development
* Listening on tcp://localhost:3000
Use Ctrl-C to stop
Started GET "/" for 127.0.0.1 at 2014-04-11 13:11:48 +0200
  ActiveRecord::SchemaMigration Load (0.2ms)  SELECT "schema_migrations".* FROM "schema_migrations"
Processing by ArticlesController#index as HTML

[3, 12] in /PathTo/project/app/controllers/articles_controller.rb
    3:
    4:   # GET /articles
    5:   # GET /articles.json
    6:   def index
    7:     byebug
=>  8:     @articles = Article.find_recent
    9:
   10:     respond_to do |format|
   11:       format.html # index.html.erb
   12:       format.json { render json: @articles }
(byebug)
```

Ahora es el momento de explorar su aplicación. Un buen lugar para comenzar es pedirle al depurador ayuda. Tipea: help

```
(byebug) help

  break      -- Sets breakpoints in the source code
  catch      -- Handles exception catchpoints
  condition  -- Sets conditions on breakpoints
  continue   -- Runs until program ends, hits a breakpoint or reaches a line
  debug      -- Spawns a subdebugger
  delete     -- Deletes breakpoints
  disable    -- Disables breakpoints or displays
  display    -- Evaluates expressions every time the debugger stops
  down       -- Moves to a lower frame in the stack trace
  edit       -- Edits source files
  enable     -- Enables breakpoints or displays
  finish     -- Runs the program until frame returns
  frame      -- Moves to a frame in the call stack
  help       -- Helps you using byebug
  history    -- Shows byebug's history of commands
  info       -- Shows several informations about the program being debugged
  interrupt  -- Interrupts the program
  irb        -- Starts an IRB session
  kill       -- Sends a signal to the current process
  list       -- Lists lines of source code
  method     -- Shows methods of an object, class or module
  next       -- Runs one or more lines of code
  pry        -- Starts a Pry session
  quit       -- Exits byebug
  restart    -- Restarts the debugged program
  save       -- Saves current byebug session to a file
  set        -- Modifies byebug settings
  show       -- Shows byebug settings
  source     -- Restores a previously saved byebug session
  step       -- Steps into blocks or methods one or more times
  thread     -- Commands to manipulate threads
  tracevar   -- Enables tracing of a global variable
  undisplay  -- Stops displaying all or some expressions when program stops
  untracevar -- Stops tracing a global variable
  up         -- Moves to a higher frame in the stack trace
  var        -- Shows variables and its values
  where      -- Displays the backtrace

(byebug)
```

Para ver las diez líneas anteriores debe escribir `list- (or l-).`

```
(byebug) l-

[1, 10] in /PathTo/project/app/controllers/articles_controller.rb
   1  class ArticlesController < ApplicationController
   2    before_action :set_article, only: [:show, :edit, :update, :destroy]
   3
   4    # GET /articles
   5    # GET /articles.json
   6    def index
   7      byebug
   8      @articles = Article.find_recent
   9
   10      respond_to do |format|
```

De esta manera puede moverse dentro del archivo y ver el código por encima de la línea donde agregó la llamada byebug. Por último, para ver dónde se encuentra en el código de nuevo se puede escribir `list=`

```
(byebug) list=

[3, 12] in /PathTo/project/app/controllers/articles_controller.rb
    3:
    4:   # GET /articles
    5:   # GET /articles.json
    6:   def index
    7:     byebug
=>  8:     @articles = Article.find_recent
    9:
   10:     respond_to do |format|
   11:       format.html # index.html.erb
   12:       format.json { render json: @articles }
(byebug)
```

### 3.3 El contexto

Cuando inicie la depuración de la aplicación, se colocará en diferentes contextos a medida que vaya a través de las diferentes partes del stack.

El depurador crea un contexto cuando se alcanza un punto de parada o un evento. El contexto tiene información sobre el programa suspendido que permite al depurador inspeccionar el stack del frame, evaluar variables desde la perspectiva del programa depurado y conocer el lugar donde se detiene el programa depurado.

En cualquier momento puede llamar al comando `backtrace` \(o su alias `where`\) para imprimir el backtrace de la aplicación. Esto puede ser muy útil para saber cómo llegaste donde estás. Si alguna vez se preguntó acerca de cómo obtuvo algún lugar en su código, entonces backtrace proporcionará la respuesta.

```
(byebug) where
--> #0  ArticlesController.index
      at /PathToProject/app/controllers/articles_controller.rb:8
    #1  ActionController::BasicImplicitRender.send_action(method#String, *args#Array)
      at /PathToGems/actionpack-5.1.0/lib/action_controller/metal/basic_implicit_render.rb:4
    #2  AbstractController::Base.process_action(action#NilClass, *args#Array)
      at /PathToGems/actionpack-5.1.0/lib/abstract_controller/base.rb:181
    #3  ActionController::Rendering.process_action(action, *args)
      at /PathToGems/actionpack-5.1.0/lib/action_controller/metal/rendering.rb:30
...
```

El frame actual está marcado con `-->`. Puede moverse a cualquier lugar que desee en este trace \(cambiando así el contexto\) utilizando el comando `frame n`, donde `n` es el número del frame especificado. Si lo hace, `byebug` mostrará su nuevo contexto.

```
(byebug) frame 2

[176, 185] in /PathToGems/actionpack-5.1.0/lib/abstract_controller/base.rb
   176:       # is the intended way to override action dispatching.
   177:       #
   178:       # Notice that the first argument is the method to be dispatched
   179:       # which is *not* necessarily the same as the action name.
   180:       def process_action(method_name, *args)
=> 181:         send_action(method_name, *args)
   182:       end
   183:
   184:       # Actually call the method associated with the action. Override
   185:       # this method if you wish to change how action methods are called,
(byebug)
```

Las variables disponibles son las mismas que si estuviera ejecutando el código línea por línea. Después de todo, eso es lo que es la depuración.

También puede utilizar los comandos `[n]` y `down [n]`  para cambiar el contexto n cuadros hacia arriba o hacia abajo del stack, respectivamente. N por defecto es uno. Up en este caso es hacia cuadros de stack de numeración superior, y hacia abajo es hacia el stack de número inferior.

### 3.4 Threads \(Hilos\)

El depurador puede enumerar, detener, reanudar y cambiar entre los subprocesos en ejecución mediante el comando `thread` \(o el abreviado `th`\). Este comando tiene un puñado de opciones:

* `thread`: muestra el hilo actual.
* `thread list`: se utiliza para enumerar todos los hilos y sus estados. El hilo actual está marcado con un signo más \(+\).
* `thread stop n`: detener el hilo n.
* `thread resume n`: reanudar el subproceso n.
* `thread switch n`: cambia el contexto de hilo actual a n.

Este comando es muy útil cuando está depurando subprocesos simultáneos y necesita verificar que no hay condiciones de competencia en su código.

### 3.5 Inspección de variables

Cualquier expresión puede ser evaluada en el contexto actual. Para evaluar una expresión, simplemente escríbala!

En este ejemplo se muestra cómo se pueden imprimir las variables de instancia definidas en el contexto actual:

```
[3, 12] in /PathTo/project/app/controllers/articles_controller.rb
    3:
    4:   # GET /articles
    5:   # GET /articles.json
    6:   def index
    7:     byebug
=>  8:     @articles = Article.find_recent
    9:
   10:     respond_to do |format|
   11:       format.html # index.html.erb
   12:       format.json { render json: @articles }

(byebug) instance_variables
[:@_action_has_layout, :@_routes, :@_request, :@_response, :@_lookup_context,
 :@_action_name, :@_response_body, :@marked_for_same_origin_verification,
 :@_config]
```

Como se habrá dado cuenta, se muestran todas las variables a las que puede acceder desde un controlador. Esta lista se actualiza dinámicamente a medida que ejecuta el código. Por ejemplo, ejecute la siguiente línea usando `next` \(aprenderá más sobre este comando más adelante en esta guía\).

```
(byebug) next

[5, 14] in /PathTo/project/app/controllers/articles_controller.rb
   5     # GET /articles.json
   6     def index
   7       byebug
   8       @articles = Article.find_recent
   9
=> 10       respond_to do |format|
   11         format.html # index.html.erb
   12        format.json { render json: @articles }
   13      end
   14    end
   15
(byebug)
```

Y luego vuelve a preguntar por las `instance_variables`:

```
(byebug) instance_variables
[:@_action_has_layout, :@_routes, :@_request, :@_response, :@_lookup_context,
 :@_action_name, :@_response_body, :@marked_for_same_origin_verification,
 :@_config, :@articles]
```

Ahora `@articles` se incluye en las variables de instancia, porque la línea que lo define se ejecutó.

> También puede entrar en el modo `irb` con el comando `irb` \(por supuesto!\). Esto iniciará una sesión `irb` dentro del contexto en el que la invocó.

El método `var` es la forma más conveniente de mostrar las variables y sus valores. Hagamos que byebug nos ayude con ella.

```
(byebug) help var

  [v]ar <subcommand>

  Shows variables and its values


  var all      -- Shows local, global and instance variables of self.
  var args     -- Information about arguments of the current scope
  var const    -- Shows constants of an object.
  var global   -- Shows global variables.
  var instance -- Shows instance variables of self or a specific object.
  var local    -- Shows local variables in current scope.
```

Esta es una excelente forma de inspeccionar los valores de las variables de contexto actuales. Por ejemplo, para comprobar que no tenemos variables locales definidas actualmente:

```
(byebug) var local
(byebug)
```

También puede inspeccionar un método de objeto de esta manera:

```
(byebug) var instance Article.new
@_start_transaction_state = {}
@aggregation_cache = {}
@association_cache = {}
@attributes = #<ActiveRecord::AttributeSet:0x007fd0682a9b18 @attributes={"id"=>#<ActiveRecord::Attribute::FromDatabase:0x007fd0682a9a00 @name="id", @value_be...
@destroyed = false
@destroyed_by_association = nil
@marked_for_destruction = false
@new_record = true
@readonly = false
@transaction_state = nil
```

También puede utilizar la pantalla para comenzar a ver las variables. Esta es una buena forma de rastrear los valores de una variable mientras la ejecución continúa.

```
(byebug) display @articles
1: @articles = nil
```

Las variables dentro de la lista mostrada se imprimirán con sus valores después de moverlo en el stack. Para detener la visualización de una variable use `undisplay n` donde n es el número de variable \(1 en el último ejemplo\).

### 3.6 Paso a paso

Ahora debe saber dónde se encuentra en el rastreo de la ejecución y ser capaz de imprimir las variables disponibles. Pero vamos a continuar y a seguir adelante con la ejecución de la aplicación.

Utilice `step` \(abreviado `s`\) para continuar ejecutando su programa hasta el siguiente punto de parada lógico y devolver el control al depurador. `next` es similar `step`, pero mientras `step` se detiene en la siguiente línea de código ejecutada, haciendo sólo un solo paso, `next` se mueve a la siguiente línea sin descender dentro de los métodos.

Por ejemplo, considere la siguiente situación:

```
Started GET "/" for 127.0.0.1 at 2014-04-11 13:39:23 +0200
Processing by ArticlesController#index as HTML

[1, 6] in /PathToProject/app/models/article.rb
   1: class Article < ApplicationRecord
   2:   def self.find_recent(limit = 10)
   3:     byebug
=> 4:     where('created_at > ?', 1.week.ago).limit(limit)
   5:   end
   6: end

(byebug)
```

Si usamos `next`, no vamos a profundizar dentro de las llamadas de método. En su lugar, byebug irá a la siguiente línea dentro del mismo contexto. En este caso, es la última línea del método actual, por lo que byebug volverá a la línea siguiente del método llamador.

```
(byebug) next
[4, 13] in /PathToProject/app/controllers/articles_controller.rb
    4:   # GET /articles
    5:   # GET /articles.json
    6:   def index
    7:     @articles = Article.find_recent
    8:
=>  9:     respond_to do |format|
   10:       format.html # index.html.erb
   11:       format.json { render json: @articles }
   12:     end
   13:   end
 
(byebug)
```

Si usamos `step` en la misma situación, byebug literalmente irá a la siguiente instrucción Ruby a ser ejecutada - en este caso, el método `week` de Active Support.

```
(byebug) step
 
[49, 58] in /PathToGems/activesupport-5.1.0/lib/active_support/core_ext/numeric/time.rb
   49:
   50:   # Returns a Duration instance matching the number of weeks provided.
   51:   #
   52:   #   2.weeks # => 14 days
   53:   def weeks
=> 54:     ActiveSupport::Duration.weeks(self)
   55:   end
   56:   alias :week :weeks
   57:
   58:   # Returns a Duration instance matching the number of fortnights provided.
(byebug)
```

Esta es una de las mejores maneras de encontrar errores en su código.

También puede utilizar el `step n` o `next n` para avanzar `n` pasos a la vez.

### 3.7 Puntos de interrupción

Un punto de interrupción hace que su aplicación se detenga cada vez que se alcanza un determinado punto en el programa. El shell depurador se invoca en esa línea.

Puede agregar puntos de interrupción de forma dinámica con el comando `break` \(o simplemente `b`\). Existen 3 formas posibles de agregar puntos de interrupción manualmente:

* `break n`: establece el punto de interrupción en la línea número n en el archivo fuente actual.
* `break file:n [if expression]`: establece el punto de interrupción en el número de línea n dentro del archivo named file. Si se da una expresión se debe evaluar a `true` para encender el depurador.
* `break class(.|\#)method [if expression]`: define el punto de interrupción en el método \(.y \# para la clase y el método de instancia respectivamente\) definidos en la clase. La expresión funciona de la misma manera que con `file:n`.

```
[4, 13] in /PathToProject/app/controllers/articles_controller.rb
    4:   # GET /articles
    5:   # GET /articles.json
    6:   def index
    7:     @articles = Article.find_recent
    8:
=>  9:     respond_to do |format|
   10:       format.html # index.html.erb
   11:       format.json { render json: @articles }
   12:     end
   13:   end
 
(byebug) break 11
Successfully created breakpoint with id 1
```

Utilice `info breakpoints` para enumerar puntos de interrupción. Si suministra un número, enumera ese punto de interrupción. De lo contrario, muestra todos los puntos de interrupción.

```
(byebug) info breakpoints
Num Enb What
1   y   at /PathToProject/app/controllers/articles_controller.rb:11
```

Para eliminar puntos de interrupción: utilice el comando `delete n` para quitar el número de punto de interrupción n. Si no se especifica ningún número, elimina todos los puntos de interrupción que están actualmente activos.

```
(byebug) delete 1
(byebug) info breakpoints
No breakpoints.
```

También puede habilitar o deshabilitar puntos de interrupción:

* `enable breakpoints [n [m [...]]]`: permite que una lista de punto de interrupción específica o todos los puntos de interrupción detengan su programa. Este es el estado predeterminado al crear un punto de interrupción.
* `disable breakpoints [n [m [...]]]`: hacer que ciertos \(o todos\) los puntos de interrupción no tengan ningún efecto en su programa.

### 3.8 Catching de las excepciones

El comando `catch exception-name (or just cat exception-name)` se puede usar para interceptar una excepción del tipo _exception-name_ cuando de otro modo no habría ningún controlador para él.

Para enumerar todos los catchpoints activos use `catch`.

### 3.9 Reanudación de la ejecución

Hay dos formas de reanudar la ejecución de una aplicación que se detiene en el depurador:

* `continue [n]`: reanuda la ejecución del programa en la dirección donde se detuvo su guión; Se anulan los puntos de interrupción establecidos en esa dirección. El argumento opcional n le permite especificar un número de línea para establecer un punto de interrupción único que se elimina cuando se alcanza ese punto de interrupción.
* `finish [n]`: ejecutar hasta que se devuelva el frame de stack seleccionado. Si no se proporciona ningún número de stack, la aplicación se ejecutará hasta que se devuelva el frame seleccionado actualmente. El frame seleccionado actualmente  comienza el stack más reciente o `0` si no se ha realizado ninguna posición de stack \(por ejemplo, arriba, abajo o stack\). Si se da un número de stack, se ejecutará hasta que se devuelva el frame especificado.

### 3.10 Edición

Dos comandos le permiten abrir código desde el depurador en un editor:

`edit [file:n]`: editar el archivo con el nombre del archivo utilizando el editor especificado por la variable de entorno EDITOR. También se puede dar una línea específica n.

### 3.11 Quitting

Para salir del depurador, utilice el comando `quit` \(abreviado a `q`\). O, escriba `q!` Para evitar el "Really quit? \(y/n\)" y salir sin condiciones.

Un simple abandono intenta terminar todos los threads en efecto. Por lo tanto su servidor será parado y usted tendrá que comenzarlo otra vez.

### 3.12 Configuración

Byebug tiene algunas opciones disponibles para ajustar su comportamiento:

    (byebug) help set

      set <setting> <value>

      Modifies byebug settings

      Boolean values take "on", "off", "true", "false", "1" or "0". If you
      don't specify a value, the boolean setting will be enabled. Conversely,
      you can use "set no<setting>" to disable them.

      You can see these environment settings with the "show" command.

      List of supported settings:

      autosave       -- Automatically save command history record on exit
      autolist       -- Invoke list command on every stop
      width          -- Number of characters per line in byebug's output
      autoirb        -- Invoke IRB on every stop
      basename       -- <file>:<line> information after every stop uses short paths
      linetrace      -- Enable line execution tracing
      autopry        -- Invoke Pry on every stop
      stack_on_error -- Display stack trace when `eval` raises an exception
      fullpath       -- Display full file names in backtraces
      histfile       -- File where cmd history is saved to. Default: ./.byebug_history
      listsize       -- Set number of source lines to list by default
      post_mortem    -- Enable/disable post-mortem mode
      callstyle      -- Set how you want method call parameters to be displayed
      histsize       -- Maximum number of commands that can be stored in byebug history
      savefile       -- File where settings are saved to. Default: ~/.byebug_save

> Puede guardar esta configuración en un archivo `.byebugrc` en su directorio personal. El depurador lee estas configuraciones globales cuando se inicia. Por ejemplo:

```ruby
set callstyle short
set listsize 25
```















