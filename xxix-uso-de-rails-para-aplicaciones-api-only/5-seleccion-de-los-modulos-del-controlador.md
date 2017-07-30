# 5- Selección de los módulos del controlador

Una aplicación de API \(con `ActionController::API`\) viene con los siguientes módulos de controladores de forma predeterminada:

* `ActionController::UrlFor`: Hace que url\_for y ayudantes similares estén disponibles.
* `ActionController::Redirecting`: Soporte a `redirect_to`.
* `AbstractController::Rendering` y `ActionController::ApiRendering`: Soporte básico para renderizado.
* `ActionController::Renderers::All:` Soporte para renderizar `:json` y amigos.
* `ActionController::ConditionalGet`: Sporte para `stale?`.
* `ActionController::BasicImplicitRender`: Se asegura de devolver una respuesta vacía, si no hay una explícita
* `ActionController::StrongParameters:` Soporte para los parámetros de lista blanca en combinación con la asignación de masa del Active Model.
* `ActionController::ForceSSL`: Soporte para `force_ssl`.
* `ActionController::DataStreaming`: Soporte para `send_file` y `send_data`.
* `AbstractController::Callbacks`: Soporte para  `before_action` y helpers similares.
* `ActionController::Rescue`: Soporte para `rescue_from`.
* `ActionController::Instrumentation`: Soporte para los hooks de instrumentación definidos por Action Controller \(consulte la guía de instrumentación para obtener más información al respecto\).
* `ActionController::ParamsWrapper`: Envuelve el hash de parámetros en un hash anidado, para que no tenga que especificar elementos raíz que envían peticiones POST por ejemplo.
* `ActionController::Head`: Soporte para devolver una respuesta sin contenido, solo encabezados

Otros complementos pueden agregar módulos adicionales. Puede obtener una lista de todos los módulos incluidos en `ActionController::API` en la consola de rails:

```
$ bin/rails c
>> ActionController::API.ancestors - ActionController::Metal.ancestors
=> [ActionController::API,
    ActiveRecord::Railties::ControllerRuntime,
    ActionDispatch::Routing::RouteSet::MountedHelpers,
    ActionController::ParamsWrapper,
    ... ,
    AbstractController::Rendering,
    ActionView::ViewPaths]
```

### 5.1 Adición de otros módulos

Todos los módulos de Action Controller conocen sus módulos dependientes, por lo que puede sentirse libre de incluir cualquier módulo en sus controladores, y todas las dependencias serán incluidas y configuradas.

Algunos módulos comunes que puede agregar:

* `AbstractController::Translation`: Soporte para los métodos de localización y traducción de` l` y `t`.
* `ActionController::HttpAuthentication::Basic` \(or Digest or Token\): Soporte para la autenticación HTTP básica, digest o token.
* `ActionView::Layouts`: Soporte para diseños al procesar.
* `ActionController::MimeResponds`: Soporte para `respond_to`.
* `ActionController::Cookies`: Soporte para cookies, que incluye soporte para cookies firmadas y cifradas. Esto requiere el middleware de cookies.

El mejor lugar para agregar un módulo está en su `ApplicationController`, pero también puede agregar módulos a controladores individuales.



