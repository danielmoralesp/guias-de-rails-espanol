# 2 ¿Por qué utilizar Rails para las API de JSON?

La primera pregunta que mucha gente tiene al pensar en construir una API de JSON usando Rails es: "¿Está usando Rails para renderizar JSON? ¿No debería usar algo como Sinatra?".

Para APIs muy simples, esto puede ser cierto. Sin embargo, incluso en aplicaciones muy pesadas en HTML, la mayoría de la lógica de una aplicación vive fuera de la capa de la vista.

La razón por la que la mayoría de la gente utiliza Rails es que proporciona un conjunto de valores predeterminados que permite a los desarrolladores ponerse en marcha rápidamente, sin tener que tomar muchas decisiones triviales.

Echemos un vistazo a algunas de las cosas que Rails ofrece fuera de la caja que siguen siendo aplicables a las aplicaciones API.

Gestionado en la capa middleware:

* Reloading: Las aplicaciones de Rails soportan la recarga transparente. Esto funciona incluso si su aplicación se vuelve grande y reiniciar el servidor para cada solicitud se convierte en algo inviable.
* Modo de desarrollo: Las aplicaciones de Rails vienen con valores predeterminados inteligentes para el modo de desarrollo, lo que hace que el desarrollo sea agradable sin comprometer el rendimiento en tiempo de producción.
* Modo de prueba: Ditto modo de desarrollo.
* Logging: las aplicaciones de Rails registran cada solicitud, con un nivel de verbosidad apropiado para el modo actual. Los logs de Rails en desarrollo incluyen información sobre el entorno de solicitud, consultas de bases de datos e información básica de rendimiento.
* Seguridad: Rails detecta y frustra ataques de spoofing de IP y maneja firmas criptográficas en una forma de ataque de sincronización. ¿No sabes lo que es un ataque de spoofing IP o un ataque de tiempo? Puedes buscarlo en internet.
* Parameter Parsing: ¿Desea especificar sus parámetros como JSON en lugar de como una cadena codificada por URL? No hay problema. Rails decodificará el JSON por usted y lo pondrá disponible en params. ¿Quiere usar parámetros anidados de codificación de URL? Eso también funciona.
* GETs condicionales: Rails maneja los encabezados de solicitud de procesamiento condicionales GET \(ETag y Last-Modified\) y devuelve los encabezados de respuesta correctos y el código de estado. Todo lo que necesitas hacer es usar el stale? Compruebe en su controlador, y Rails manejará todos los detalles del HTTP por usted.
* HEAD requests: Rails convertirá transparentemente las peticiones HEAD en GET, y devolverá sólo los encabezados en la salida. Esto hace que HEAD funcione de manera fiable en todas las API de Rails.

Si bien obviamente podría construir esto en términos de middleware de rack existentes, esta lista demuestra que el stack de middleware de Rails predeterminado proporciona mucho valor, incluso si está "generando JSON".

Gestionado en la capa Action Pack:

* Enrutamiento Resourceful: si está creando una API JSON RESTful, desea utilizar el enrutador de Rails. El mapeo limpio y convencional de HTTP a controladores significa no tener que pasar tiempo pensando en cómo modelar su API en términos de HTTP.
* Generación de URLs: La otra cara del enrutamiento es la generación de las URLs. Una buena API basada en HTTP incluye URLs \(consulte el [GitHub Gist API](https://developer.github.com/v3/gists/) para ver un ejemplo\).
* Encabezado y Redirección de Respuestas: `head: no_content` y `redirect_to user_url(current_user)` vienen a mano. Claro, puede añadir manualmente los encabezados de respuesta, pero ¿para qué?
* Almacenamiento en caché: Rails proporciona caché de página, acción y fragmentos. El almacenamiento en caché de fragmentos es especialmente útil cuando se construye un objeto JSON anidado.
* Autenticación básica, de resumen y de token: Rails viene con soporte para tres tipos de autenticación HTTP.
* Instrumentación: Rails tiene una API de instrumentación que activa gestores registrados para una variedad de eventos, como procesamiento de acciones, envío de un archivo o datos, redirección y consultas de base de datos. La carga útil de cada evento viene con información relevante \(para el evento de procesamiento de acciones, la carga incluye el controlador, la acción, los parámetros, el formato de solicitud, el método de solicitud y la ruta completa de la solicitud\).
* Generadores: A menudo es útil generar un recurso y obtener su modelo, controlador, stubs de prueba y rutas creadas para usted en un solo comando para ajustes adicionales. Igual para migraciones y otras.
* Plugins: Muchas bibliotecas de terceros vienen con soporte para Rails que reducen o eliminan el costo de configurar y pegar la biblioteca y el framework web. Esto incluye cosas como reemplazar generadores predeterminados, agregar tareas de Rake y honrar las opciones de Rails \(como el registrador y el back-end de caché\).

Por supuesto, el proceso de arranque de Rails también encola todos los componentes registrados. Por ejemplo, el proceso de arranque de Rails utiliza su archivo `config/database.yml` al configurar Active Record.

La versión corta es: es posible que no haya pensado en qué partes de Rails siguen siendo aplicables incluso si se elimina la capa de vista, pero la respuesta resulta ser la mayor parte.





