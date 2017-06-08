# 2- Creación de respuestas

Desde el punto de vista del controlador, hay tres maneras de crear una respuesta `HTTP`:

* Llamar a `render` para crear una respuesta completa para enviar de nuevo al navegador
* Llamar a `redirect_to` para enviar un código de estado de redireccionamiento `HTTP` al navegador
* Llamar a `head` para crear una respuesta que consiste únicamente en encabezados `HTTP` para enviar de nuevo al navegador



