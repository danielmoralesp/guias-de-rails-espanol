# 10- Objetos de solicitud y respuesta

En cada controlador hay dos métodos de acceso que apuntan a la solicitud y los objetos de respuesta asociados con el ciclo de solicitud que está actualmente en ejecución. El método de solicitud contiene una instancia de `ActionDispatch::Request` y el método de respuesta devuelve un objeto de respuesta que representa lo que se va a devolver al cliente.

### 10.1 El objeto Request

El objeto request contiene mucha información útil acerca de la solicitud que viene desde el cliente. Para obtener una lista completa de los métodos disponibles, consulte la documentación de la API de Rails y la documentación en Rack. Entre las propiedades a las que puede acceder en este objeto se encuentran:

| Propiedad de la solicitud | Propósito |
| :--- | :--- |
| host | El nombre de host utilizado para esta solicitud. |
| domain\(n=2\) | Los primeros n segmentos del nombre de host, comenzando desde la derecha \(el TLD\). |
| format | El tipo de contenido solicitado por el cliente. |
| method | El método HTTP utilizado para la solicitud. |
| get?, post?, path?, put?, delete?, head? | Devuelve true si el método HTTP es GET / POST / PATCH / PUT / DELETE / HEAD |
| headers | Devuelve un hash que contiene los encabezados asociados con la petición. |
| port | Número de puerto \(número entero\) utilizado para la solicitud |
| protocol | Devuelve una cadena que contiene el protocolo utilizado más "://", por ejemplo "http://". |
| query\_string | La cadena de consulta forma parte de la URL, es decir, todo después de "?". |
| remote\_ip | La dirección IP del cliente. |
| url | URL completa utilizada para la solicitud. |

#### 10.1.1 path\_parameters, query\_parameters y request\_parameters

Rails recopila todos los parámetros enviados junto con la solicitud en el hash de parámetros, ya sea que se envíen como parte de la cadena de consulta o el cuerpo de la publicación. El objeto de la petición tiene tres accesores que le dan acceso a estos parámetros dependiendo de su procedencia. El hash `query_parameters` contiene parámetros que se enviaron como parte de la cadena de consulta mientras que el hash `request_parameters` contiene los parámetros enviados como parte del cuerpo de la publicación. El hash `path_parameters` contiene parámetros que fueron reconocidos por el enrutamiento como parte del camino que conduce a este controlador y acción en particular.

### 10.2 El objeto Response

El objeto de respuesta no suele usarse directamente, sino que se construye durante la ejecución de la acción y el procesamiento de los datos que se envían al usuario, pero a veces, como en un filtro `after`, puede ser útil acceder a la respuesta directamente. Algunos de estos métodos de acceso también tienen setters, lo que le permite cambiar sus valores. Para obtener una lista completa de los métodos disponibles, consulte la documentación de la API de Rails y la documentación de Rack.

| Propiedad de la respuesta | Propósito |
| :--- | :--- |
| body | Es la cadena de datos que se devuelven al cliente. Esto es frecuentemente HTML. |
| status | El código de estado HTTP de la respuesta, como 200 para una solicitud satisfactoria o 404 para el archivo no encontrado |
| location | La URL a la que se está redireccionando el cliente, si lo hay. |
| content\_type | El tipo de contenido de la respuesta. |
| charset | El conjunto de caracteres que se utiliza para la respuesta. El valor predeterminado es "utf-8". |
| headers | Encabezados utilizados para la respuesta. |

#### 10.2.1 Configuración de encabezados personalizados

Si desea establecer encabezados personalizados para una respuesta, entonces `response.headers` es el lugar para hacerlo. El atributo headers es un hash que asigna nombres de encabezado a sus valores, y Rails establecerá algunos de ellos automáticamente. Si desea agregar o cambiar un encabezado, sólo tiene que asignarlo a response.headers de esta manera:

```ruby
response.headers["Content-Type"] = "application/pdf"
```

Nota: en el caso anterior tendría más sentido usar directamente el setter `content_type`.

