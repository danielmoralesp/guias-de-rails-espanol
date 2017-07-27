# 1- Una introducción a Ajax

Para entender Ajax, primero debes entender lo que hace normalmente un navegador web.

Cuando escribes `http://localhost:3000` en la barra de direcciones de su navegador y pulsa 'Ir', el navegador \(su 'cliente'\) hace una petición al servidor. Analiza la respuesta y luego busca todos los recursos asociados, como archivos JavaScript, hojas de estilo e imágenes. A continuación, monta la página. Si hace clic en un vínculo, realiza el mismo proceso: busque la página, obtenga los recursos, muéstrelos todos, muéstrele los resultados. Esto se conoce como el "ciclo de respuesta de solicitud".

JavaScript también puede hacer peticiones al servidor y analizar la respuesta. También tiene la capacidad de actualizar la información de la página. Combinando estas dos potencias, un escritor de JavaScript puede hacer una página web que pueda actualizar sólo ciertas partes del mismo, sin necesidad de obtener todos los datos de la página completa desde el servidor. Esta es una técnica poderosa que llamamos Ajax.

Rails se envía con CoffeeScript de forma predeterminada y, por lo tanto, el resto de los ejemplos de esta guía aparecerán en CoffeeScript. Todas estas lecciones, por supuesto, se aplican a la JavaScript vanila  también.

Como ejemplo, he aquí un código de CoffeeScript que hace una solicitud de Ajax usando la biblioteca jQuery:

```js
$.ajax(url: "/test").done (html) ->
  $("#results").append html
```

Este código recupera los datos de "`/test`", y luego agrega el resultado al `div` con un `id` `results`.

Rails proporciona un poco de soporte incorporado para la construcción de páginas web con esta técnica. Rara vez tiene que escribir este código usted mismo. El resto de esta guía le mostrará cómo Rails puede ayudarle a escribir sitios web de esta manera, pero todo se basa en esta técnica bastante simple.



