# 3- La configuración básica

Si está creando una aplicación de Rails que será un servidor API en primer lugar, puede comenzar con un subconjunto más limitado de Rails y agregar funciones según sea necesario.

### 3.1 Creación de una nueva aplicación

Puede generar una nueva aplicación de Rails api:

```ruby
$ rails new my_api --api
```

Esto hará tres cosas principales por usted:

* Configura su aplicación para comenzar con un conjunto de middleware más limitado de lo normal. Específicamente, no incluirá ningún middleware que resulte útil para aplicaciones de navegador \(como el soporte de cookies\) de forma predeterminada.
* Hace que `ApplicationController` herede de `ActionController::API` en lugar de `ActionController::Base`. Al igual que con el middleware, esto dejará fuera cualquier módulo de Action Controller que provea funcionalidades usadas principalmente por aplicaciones de navegador.
* Configure los generadores para omitir la generación de vistas, helpers y resources al generar un nuevo recurso.

### 3.2 Cambio de una aplicación existente

Si desea tomar una aplicación existente y convertirla en una API, lea los pasos siguientes.

En `config/application.rb`, agregue la siguiente línea en la parte superior de la definición de la clase `Application`:

```ruby
config.api_only = true
```

En `config/environments/development.rb`, configure `config.debug_exception_response_format` para configurar el formato utilizado en las respuestas cuando se producen errores en el modo de desarrollo.

Para renderizar una página HTML con información de depuración, utilice el valor `:default`.

```ruby
config.debug_exception_response_format = :default
```

Para hacer que la información de depuración mantenga el formato de respuesta, utilice el valor `:api`.

```ruby
config.debug_exception_response_format = :api
```

De forma predeterminada, `config.debug_exception_response_format` se establece en `:api`, cuando `config.api_only` se establece en `true`.

Por último, dentro de `app/controllers/application_controller.rb`, en lugar de:

```ruby
class ApplicationController < ActionController::Base
end
```

hacer:

```ruby
class ApplicationController < ActionController::API
end
```







