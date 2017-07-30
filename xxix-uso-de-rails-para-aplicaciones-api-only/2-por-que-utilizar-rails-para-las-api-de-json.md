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



