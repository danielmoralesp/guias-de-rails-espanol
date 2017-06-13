# 1- ¿Qué hace un controlador?

Action Controller es la C en MVC. Después de que el enrutador ha determinado qué controlador utilizar para una solicitud, el controlador es responsable de dar sentido a la solicitud y producir la salida adecuada. Por suerte, el controlador de acción hace la mayor parte del trabajo duro por usted y utiliza las convenciones inteligentes para hacer esto lo más directo posible.

Para la mayoría de las aplicaciones convencionales RESTful, el controlador recibe la solicitud o request \(esto es invisible para usted como desarrollador\), busca o guarda datos de un modelo y utiliza una vista para crear una salida HTML. Si su controlador necesita hacer cosas un poco diferentes, eso no es un problema, porque esta es sólo la forma más común en que un controlador  trabaja.

Por lo tanto, un controlador puede considerarse como un intermediario entre los modelos y las vistas. Hace que los datos del modelo estén disponibles para que la vista pueda mostrar esos datos al usuario, y guarda o actualiza los datos que ingresa un usuario en el modelo.

> Para obtener más detalles sobre el proceso de enrutamiento, consulte Rails Routing.



