# 14- Rescue

Lo más probable es que su aplicación va a contener errores o lanzar una excepción que debe tratarse. Por ejemplo, si el usuario sigue un vínculo a un recurso que ya no existe en la base de datos, Active Record lanzará la excepción `ActiveRecord::RecordNotFound`.

El control de excepciones por defecto de Rails muestra un mensaje "500 Server Error" para todas las excepciones. Si la solicitud se realizó localmente, un buen rastreo y una cierta información agregada se muestra para que pueda averiguar qué salió mal y tratar con él. Si la solicitud era remota, Rails sólo mostrará un simple mensaje de error "500 Server Error" al usuario, o un "404 Not Found" si hubo un error de enrutamiento o un registro no se pudo encontrar. A veces es posible que desee personalizar cómo se detectan estos errores y cómo se muestran al usuario. Hay varios niveles de manejo de excepciones disponibles en una aplicación de Rails:

### 14.1 Las plantillas predeterminadas 500 y 404

De forma predeterminada, una aplicación de producción mostrará un mensaje de error 404 o 500, en el entorno de desarrollo se generarán todas las excepciones no controladas. Estos mensajes están contenidos en archivos HTML estáticos en la carpeta pública, en `404.html` y `500.html` respectivamente. Puede personalizar estos archivos para agregar información adicional y estilos, pero recuerde que son HTML estático; Es decir, no puede utilizar `ERB`, `SCSS`, `CoffeeScript` o diseños para ellos.

### 14.2 rescue\_from

Si quieres hacer algo más elaborado al detectar errores, puedes usar `rescue_from`, que maneja excepciones de cierto tipo \(o varios tipos\) en un controlador completo y sus subclases.

Cuando se produce una excepción que es detectada por una directiva `rescue_from`, el objeto de excepción se pasa al manejador \(handler\). El manejador \(handler\) puede ser un método o un objeto `Proc` pasado a la opción `:with`. También puede utilizar un bloque directamente en lugar de un objeto `Proc` explícito.

A continuación, le indicamos cómo puede utilizar `rescue_from` para interceptar todos los errores de `ActiveRecord::RecordNotFound` y hacer algo con ellos.

```ruby
class ApplicationController < ActionController::Base
  rescue_from ActiveRecord::RecordNotFound, with: :record_not_found
 
  private
 
    def record_not_found
      render plain: "404 Not Found", status: 404
    end
end
```

Por supuesto, este ejemplo es todo menos elaborado y no mejora la manipulación de excepciones por defecto en absoluto, pero una vez que puedas capturar todas esas excepciones eres libre de hacer lo que quieras con ellas. Por ejemplo, puede crear clases de excepción personalizadas que se lanzarán cuando un usuario no tenga acceso a una determinada sección de su aplicación:

```ruby
class ApplicationController < ActionController::Base
  rescue_from User::NotAuthorized, with: :user_not_authorized
 
  private
 
    def user_not_authorized
      flash[:error] = "You don't have access to this section."
      redirect_back(fallback_location: root_path)
    end
end
 
class ClientsController < ApplicationController
  # Check that the user has the right authorization to access clients.
  before_action :check_authorization
 
  # Note how the actions don't have to worry about all the auth stuff.
  def edit
    @client = Client.find(params[:id])
  end
 
  private
 
    # If the user is not authorized, just throw the exception.
    def check_authorization
      raise User::NotAuthorized unless current_user.admin?
    end
end
```

> No debe hacer la excepción `rescue_from`  o `rescue_from StandardError` a menos que tenga una razón en particular, ya que causará efectos secundarios graves \(por ejemplo, no podrá ver detalles de excepción y rastreos durante el desarrollo\).

> Cuando se ejecuta en el entorno de producción, todos los errores `ActiveRecord::RecordNotFound` generan la página de error 404. A menos que necesites un comportamiento personalizado, no necesitas manejarlo

> Ciertas excepciones sólo se pueden rescatar de la clase ApplicationController, ya que se generan antes de que el controlador se inicialice y se ejecute la acción.



