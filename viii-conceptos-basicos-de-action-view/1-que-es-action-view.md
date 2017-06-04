# 1 ¿Qué es Action View?

En Rails, las solicitudes web son manejadas por Action Controller y Action View. Normalmente, Action Controller se ocupa de comunicarse con la base de datos y realizar acciones CRUD cuando sea necesario. Entonces, Action View es responsable de compilar la respuesta.

Los templates de Action View se escriben utilizando Ruby incrustado en etiquetas mezcladas con HTML. Para evitar el desorden de las plantillas con el código boilerplate, un número de clases `helper` proporcionan un comportamiento común para los formularios, fechas y strings. También es fácil agregar nuevos helpers a su aplicación a medida que evoluciona.

> Algunas características de Action View están vinculadas a Active Record, pero eso no significa que Action View depende de Active Record. Action View es un paquete independiente que se puede utilizar con cualquier tipo de bibliotecas Ruby.



