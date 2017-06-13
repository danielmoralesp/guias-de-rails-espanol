# 3- Métodos y acciones

Un controlador es una clase Ruby que hereda de `ApplicationController` y tiene métodos como cualquier otra clase. Cuando su aplicación recibe una solicitud, el enrutamiento determinará qué controlador y acción ejecutar, entonces Rails crea una instancia de ese controlador y ejecuta el método con el mismo nombre que la acción.

```ruby
class ClientsController < ApplicationController
  def new
  end
end
```

Por ejemplo, si un usuario va a `/clients/new` en su aplicación para agregar un nuevo cliente, Rails creará una instancia de `ClientsController` y llamará a su nuevo método. Tenga en cuenta que el método vacío del ejemplo anterior funcionaría correctamente porque Rails procesará por defecto la vista `new.html.erb`, a menos que la acción indique lo contrario. El nuevo método podría poner a disposición de la vista una variable de instancia `@client` creando un nuevo Cliente:

```ruby
def new
  @client = Client.new
end
```

La Guía de Layouts y Rendering explica esto con más detalle.

`ApplicationController` hereda de `ActionController::Base`, que define una serie de métodos útiles. Esta guía cubre algunos de estos, pero si tienes curiosidad de ver lo que hay ahí, puedes verlos todos en la documentación de la API o en la fuente misma.

Sólo los métodos públicos son llamados como acciones. Es una buena práctica reducir la visibilidad de los métodos \(privados o protegidos\) que no pretenden ser acciones, como los métodos auxiliares o los filtros.

