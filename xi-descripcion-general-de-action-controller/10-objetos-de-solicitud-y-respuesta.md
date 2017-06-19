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



