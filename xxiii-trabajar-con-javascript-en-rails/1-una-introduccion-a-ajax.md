# 1- Una introducción a Ajax

Para entender Ajax, primero debes entender lo que hace normalmente un navegador web.

Cuando escribes `http://localhost:3000` en la barra de direcciones de su navegador y pulsa 'Ir', el navegador \(su 'cliente'\) hace una petición al servidor. Analiza la respuesta y luego busca todos los recursos asociados, como archivos JavaScript, hojas de estilo e imágenes. A continuación, monta la página. Si hace clic en un vínculo, realiza el mismo proceso: busque la página, obtenga los recursos, muéstrelos todos, muéstrele los resultados. Esto se conoce como el "ciclo de respuesta de solicitud".

JavaScript también puede hacer peticiones al servidor y analizar la respuesta. También tiene la capacidad de actualizar la información en la página. Combinando estas dos potencias, un escritor de JavaScript puede hacer una página web que puede actualizar sólo partes del mismo, sin necesidad de obtener los datos de la página completa desde el servidor. Esta es una técnica poderosa que llamamos Ajax.

