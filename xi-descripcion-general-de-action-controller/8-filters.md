# 8- Filters

Los filtros son métodos que se ejecutan `before` "antes",  `after` "después" o `around` "alrededor" de una acción del controlador.

Los filtros se heredan, por lo que si se establece un filtro en `ApplicationController`, se ejecutará en cada controlador de la aplicación.

Un filtro "before" puede detener el ciclo de una solicitud. Un filtro "before" común es aquel que requiere que un usuario esté conectado para que se ejecute una acción. Puede definir el método de filtro de esta manera:

```ruby
class ApplicationController < ActionController::Base
  before_action :require_login
 
  private
 
  def require_login
    unless logged_in?
      flash[:error] = "You must be logged in to access this section"
      redirect_to new_login_url # halts request cycle
    end
  end
end
```

El método simplemente almacena un mensaje de error en el `flash` y redirige al formulario de inicio de sesión si el usuario no ha iniciado sesión. Si un filtro "before" hace o redirecciona, la acción no se ejecutará. Si hay filtros adicionales programados para ejecutarse después de ese filtro, también se cancelarán.

En este ejemplo, el filtro se agrega a `ApplicationController` y, por lo tanto, todos los controladores de la aplicación lo heredan. Esto hará que todo en la aplicación requiera que el usuario esté conectado para poder usarlo. Por razones obvias \(el usuario no podría acceder a nada en la aplicación hasta iniciar sesión en primer lugar!\), No todos los controladores o acciones deben requerir esto. Puede evitar que este filtro se ejecute antes de acciones particulares con `skip_before_action`:

```ruby
class LoginsController < ApplicationController
  skip_before_action :require_login, only: [:new, :create]
end
```

Ahora, las acciones `new` y `create` de `LoginsController` funcionarán como antes sin requerir que el usuario esté conectado. La opción `:only` se usa para omitir este filtro solo para estas acciones, y también hay una opción `:except` que funciona de manera opuesta. Estas opciones también se pueden utilizar al agregar filtros, por lo que puede agregar un filtro que sólo se ejecuta para acciones seleccionadas en primer lugar.

### 8.1 Filtros "after" y "around"

Además de los filtros "before", también puede ejecutar filtros después de que se haya ejecutado una acción o ambos \(antes y después\). 

Los filtros "after" son similares a los filtros "before", pero debido a que la acción ya se ha ejecutado tienen acceso a los datos de respuesta que están a punto de ser enviados por el cliente. Obviamente, los filtros "after" no pueden detener la ejecución de la acción.

Los filtros "around" son responsables de ejecutar sus acciones asociadas por yielding, similar a cómo funcionan los middlewares Rack.

Por ejemplo, en un sitio web en el que los cambios tienen un flujo de trabajo de aprobación, un administrador puede obtener una vista previa de ellos fácilmente, y solo debe aplicarlos en una transacción:

```ruby
class ChangesController < ApplicationController
  around_action :wrap_in_transaction, only: :show
 
  private
 
  def wrap_in_transaction
    ActiveRecord::Base.transaction do
      begin
        yield
      ensure
        raise ActiveRecord::Rollback
      end
    end
  end
end
```

Tenga en cuenta que un filtro "around" también envuelve el rendering. En particular, si en el ejemplo anterior, la vista se lee desde la base de datos \(por ejemplo, a través de un ámbito\), lo hará dentro de la transacción y, de este modo, presentará los datos para obtener una vista previa.

Usted puede elegir no ceder y construir la respuesta usted mismo, en cuyo caso la acción no se ejecutará.

### 8.2 Otras formas de utilizar filtros

Aunque la forma más común de usar filtros es creando métodos privados y usando `* _action` para agregarlos, hay otras dos maneras de hacer lo mismo.

La primera es usar un bloque directamente con los métodos `* _action`. El bloque recibe el controlador como un argumento. El filtro `require_login` de arriba podría ser reescrito para usar un bloque:

```ruby
class ApplicationController < ActionController::Base
  before_action do |controller|
    unless controller.send(:logged_in?)
      flash[:error] = "You must be logged in to access this section"
      redirect_to new_login_url
    end
  end
end
```

Tenga en cuenta que el filtro en este caso utiliza `send` porque el metodo `logged_in?` es privado y el filtro no se ejecuta en el ámbito del controlador. Esta no es la forma recomendada de implementar este filtro en particular, pero en casos más simples podría ser útil.

La segunda forma es usar una clase \(en realidad, cualquier objeto que responda a los métodos correctos lo hará\) para manejar el filtrado. Esto es útil en casos que son más complejos y no se pueden implementar de una manera legible y reutilizable usando los otros dos métodos. Como ejemplo, podría volver a escribir el filtro de inicio de sesión de nuevo para usar una clase:

```ruby
class ApplicationController < ActionController::Base
  before_action LoginFilter
end
 
class LoginFilter
  def self.before(controller)
    unless controller.send(:logged_in?)
      controller.flash[:error] = "You must be logged in to access this section"
      controller.redirect_to controller.new_login_url
    end
  end
end
```

Una vez más, este no es un ejemplo ideal para este filtro, ya que no se ejecuta en el ámbito del controlador, pero obtiene el controlador pasado como un argumento. La clase filter debe implementar un método con el mismo nombre que el filtro, por lo que para el filtro `before_action` la clase debe implementar un método `before` y así sucesivamente. El método `around` debe ceder para ejecutar la acción.

