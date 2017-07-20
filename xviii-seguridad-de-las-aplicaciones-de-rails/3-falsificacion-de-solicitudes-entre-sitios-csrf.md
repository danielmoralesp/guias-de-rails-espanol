# 3- Falsificación de solicitudes entre sitios \(CSRF\)

Este método de ataque funciona mediante la inclusión de código malicioso o un link en una página que accede a una aplicación web que el usuario cree que se ha autenticado. Si la sesión para esa aplicación web no ha expirado, un atacante puede ejecutar comandos no autorizados.

![](/assets/Captura de pantalla 2017-07-20 a las 9.48.03 a.m..png)

En el capítulo de sesiones has aprendido que la mayoría de las aplicaciones de Rails utilizan sesiones basadas en cookies. O almacenan el ID de sesión en la cookie y tienen un hash de sesión en el lado del servidor, o el hash de sesión completo está en el lado del cliente. En cualquier caso, el navegador enviará automáticamente a lo largo de la cookie en cada solicitud a un dominio, si puede encontrar una cookie para ese dominio. El punto polémico es que si la solicitud proviene de un sitio de un dominio diferente, también enviará la cookie. Comencemos con un ejemplo:

* Bob navega por un tablero de mensajes y ve una entrada de un hacker donde hay un elemento de imagen HTML creado. El elemento hace referencia a un comando en la aplicación de administración de proyectos de Bob, en lugar de un archivo de imagen: `<img src = "http://www.webapp.com/project/1/destroy">`
* La sesión de Bob en `www.webapp.com` todavía está viva, porque no se cerró la sesión hace unos minutos.
* Al ver el post, el navegador encuentra una etiqueta de imagen. Intentará cargar la imagen sospechada de `www.webapp.com`. Como se explicó anteriormente, también enviará a lo largo de la cookie con el ID de sesión válido.
* La aplicación web en `www.webapp.com `verifica la información del usuario en el hash de sesión correspondiente y destruye el proyecto con el `ID` 1. A continuación, devuelve una página de resultados que es un resultado inesperado para el navegador, por lo que no mostrará la imagen.
* Bob no se da cuenta del ataque, pero pocos días después descubre que el proyecto número uno ha desaparecido.

Es importante notar que la imagen real o el enlace no necesariamente tiene que estar situado en el dominio de la aplicación web, puede estar en cualquier lugar, en un foro, una publicación de blog o un correo electrónico.

CSRF aparece muy raramente en CVE \(Common Vulnerabilities and Exposures\) - menos del 0,1% en 2006 - pero es realmente un "gigante dormido" \[Grossman\]. Esto contrasta fuertemente con los resultados en muchos contratos de seguridad - CSRF es un importante problema de seguridad.

### 3.1 Contramedidas CSRF

> Primero, como es requerido por el W3C, utilice GET y POST apropiadamente. En segundo lugar, un token de seguridad en solicitudes no GET que protegerán su aplicación de CSRF.

El protocolo HTTP básicamente proporciona dos tipos principales de solicitudes: GET y POST \(y más, pero no son compatibles con la mayoría de los navegadores\). El World Wide Web Consortium \(W3C\) proporciona una lista de comprobación para elegir HTTP GET o POST:

Utilice GET si:

* La interacción se parece más a una pregunta \(es decir, es una operación segura como una consulta, una operación de lectura o una búsqueda\).

Utilice POST si:

* La interacción es más bien una orden, o
* La interacción cambia el estado del recurso de una manera que el usuario percibiría \(por ejemplo, una suscripción a un servicio\), o
* El usuario es responsable por los resultados de la interacción.

Si su aplicación web es RESTful, podría estar acostumbrado a verbos HTTP adicionales, como `PATCH`, `PUT` o `DELETE`. Sin embargo, la mayoría de los navegadores web de hoy en día no los admiten, sólo `GET` y `POST`. Rails utiliza un campo `_method` oculto para manejar esta barrera.

Las solicitudes POST también se pueden enviar automáticamente. En este ejemplo, el enlace www.harmless.com se muestra como el destino en la barra de estado del navegador. Pero en realidad ha creado dinámicamente un nuevo formulario que envía una solicitud POST.

```html
<a href="http://www.harmless.com/" onclick="
  var f = document.createElement('form');
  f.style.display = 'none';
  this.parentNode.appendChild(f);
  f.method = 'POST';
  f.action = 'http://www.example.com/account/destroy';
  f.submit();
  return false;">To the harmless survey</a>
```

O el atacante coloca el código en el controlador de eventos `onmouseover` de una imagen:

```html
<img src="http://www.harmless.com/img" width="400" height="400" onmouseover="..." />
```

Hay muchas otras posibilidades, como usar una etiqueta `<script>` para hacer una solicitud entre sitios a una URL con una respuesta JSONP o JavaScript. La respuesta es código ejecutable en el que el atacante puede encontrar una manera de ejecutar, posiblemente extrayendo datos sensibles. Para protegerse contra esta fuga de datos, debemos rechazar las etiquetas `<script>` entre sitios. Las solicitudes de Ajax, sin embargo, obedecen la política del mismo origen del navegador \(solo su propio sitio puede iniciar `XmlHttpRequest`\) para que podamos permitirles devolver respuestas JavaScript de forma segura.

Nota: no podemos distinguir el origen de una etiqueta `<script>`, ya sea una etiqueta en su propio sitio o en otro sitio malicioso, por lo que debemos bloquear todos los` <script>` en el board, aunque sea realmente un origen seguro, como el Script de su propio sitio. En estos casos, omita explícitamente la protección CSRF en las acciones que sirven JavaScript destinado a una etiqueta `<script>`.

Para protegernos de todas las demás solicitudes falsificadas, introducimos un token de seguridad requerido que nuestro sitio sabe, pero otros sitios no lo saben. Incluimos el token de seguridad en las solicitudes y lo verificamos en el servidor. Esta es una línea única en su controlador de aplicaciones y es el valor predeterminado para las aplicaciones de Rails recién creadas:

```ruby
protect_from_forgery with: :exception
```

Esto incluirá automáticamente un token de seguridad en todos los formularios y las solicitudes Ajax generadas por Rails. Si el token de seguridad no coincide con lo esperado, se lanzará una excepción.

> De forma predeterminada, Rails incluye un adaptador de secuencias de comandos discreto que agrega un encabezado denominado `X-CSRF-Token` con el token de seguridad en cada llamada Ajax no GET. Sin este encabezado, las solicitudes de Ajax que no sean GET no serán aceptadas por Rails. Cuando utilice otra biblioteca para realizar llamadas Ajax, es necesario agregar el token de seguridad como encabezado predeterminado para las llamadas Ajax en su biblioteca. Para obtener el token, eche un vistazo a la etiqueta `<meta name='csrf-token' content='THE-TOKEN'>` impresa por `<%= csrf_meta_tags%>` en la vista de la aplicación.

Es común utilizar cookies persistentes para almacenar información de usuario, con `cookies.permanent`, por ejemplo. En este caso, las cookies no se borrarán y la protección `CSRF` no será efectiva. Si está usando un almacenamiento de cookies diferente a la sesión para obtener esta información, debe manejar lo que debe hacer con ella:

```ruby
rescue_from ActionController::InvalidAuthenticityToken do |exception|
  sign_out_user # Example method that will destroy the user cookies
end
```

El método anterior se puede colocar en `ApplicationController` y se llamará cuando un token CSRF no esté presente o sea incorrecto en una solicitud que no sea GET.

Tenga en cuenta que las vulnerabilidades de _XSS \(cross-site scripting\)_ pasan por alto todas las protecciones _CSRF_. _XSS_ proporciona al atacante acceso a todos los elementos de una página, para que puedan leer el token de seguridad _CSRF_ desde un formulario o enviar directamente el formulario. Más información sobre _XSS_ más adelante.



