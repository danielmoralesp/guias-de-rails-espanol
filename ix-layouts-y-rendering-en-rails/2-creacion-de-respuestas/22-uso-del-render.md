# 2.2 Uso del render

En la mayoría de los casos, el método `ActionController::Base#render` hace el trabajo pesado de renderizar el contenido de su aplicación para su uso en el navegador. Hay una variedad de maneras de personalizar el comportamiento de `render`. Puede renderizar la vista predeterminada para una plantilla de Rails, o una plantilla específica, o un archivo, o código inline, o nada en absoluto. Puede procesar texto, `JSON` o `XML`. También puede especificar el tipo de contenido o el estado `HTTP` de la respuesta renderizada.

Si desea ver los resultados exactos de una llamada a `render` sin necesidad de inspeccionarla en un navegador, puede llamar a `render_to_string`. Este método toma exactamente las mismas opciones que `render`, pero devuelve una cadena en lugar de enviar una respuesta al navegador.

