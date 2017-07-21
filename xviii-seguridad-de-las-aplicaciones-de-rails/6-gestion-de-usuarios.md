# 6- Gestión de usuarios

> Casi todas las aplicaciones web tienen que lidiar con la autorización y la autenticación. En lugar de hacer su propio código, es aconsejable utilizar plug-ins comunes. Pero también, mantenerlos al día. Algunas precauciones adicionales pueden hacer que su aplicación sea aún más segura.

Existen varios complementos de autenticación disponibles para Rails. Los buenos, como `devise` y `authlogic`, almacenan sólo contraseñas encriptadas, no contraseñas de texto sin formato. En Rails 3.1 puede utilizar el método `has_secure_password` incorporado que tiene características similares.

Cada nuevo usuario recibe un código de activación para activar su cuenta cuando recibe un correo electrónico con un enlace. Después de activar la cuenta, las columnas `activation_code` se establecerán en NULL en la base de datos. Si alguien solicitó una URL como ésta, se registraría como el primer usuario activado que se encuentre en la base de datos \(y es probable que se trate del administrador\):

```ruby
http://localhost:3006/user/activate
http://localhost:3006/user/activate?id=
```

Esto es posible porque en algunos servidores, de esta manera el parámetro `id`, como en `params[:id]`, sería `nil`. Sin embargo, aquí está el buscador de la acción de activación:

```ruby
User.find_by_activation_code(params[:id])
```

Si el parámetro era `nil`, la consulta SQL resultante será

```ruby
SELECT * FROM users WHERE (users.activation_code IS NULL) LIMIT 1
```

Y así encontró al primer usuario en la base de datos, lo devolvió y lo registró. Puede obtener más información sobre él en esta [entrada de blog](https://rorsecurity.info/journal/2007/10/28/restful_authentication-login-security.html). Es aconsejable actualizar sus plug-ins de vez en cuando. Además, puede revisar su aplicación para encontrar más fallas como esta.

### 6.1 Ataques de fuerza bruta en accounts

> Los ataques de fuerza bruta en las cuentas son ataques de prueba y error en las credenciales de inicio de sesión. Abandonarlos con mensajes de error más genéricos y posiblemente necesite introducir un CAPTCHA.

Una lista de nombres de usuario para su aplicación web puede ser mal utilizada para forzar las contraseñas, porque la mayoría de la gente no usa contraseñas sofisticadas. La mayoría de las contraseñas son una combinación de palabras del diccionario y posiblemente números. Así armado con una lista de nombres de usuario y un diccionario, un programa automático puede encontrar la contraseña correcta en cuestión de minutos.

Debido a esto, la mayoría de las aplicaciones web mostrará un mensaje de error genérico "nombre de usuario o contraseña no correcta", si una de ellas no es correcta. Si dice que "no se ha encontrado el nombre de usuario que ha introducido", un atacante podría compilar automáticamente una lista de nombres de usuario.

Sin embargo, lo que la mayoría de los diseñadores de aplicaciones web descuidan, son las páginas de "olvidé mi contraseña". Estas páginas admiten a menudo que el nombre de usuario introducido o la dirección de correo electrónico no se ha encontrado. Esto permite a un atacante compilar una lista de nombres de usuario y forzar las cuentas.

Con el fin de mitigar estos ataques, mostrar un mensaje de error genérico en las páginas de "olvidé mi contraseña", también. Además, puede requerir introducir un CAPTCHA después de un número de inicios de sesión fallidos de una determinada dirección IP. Tenga en cuenta, sin embargo, que esto no es una solución a prueba de balas contra programas automáticos, porque estos programas pueden cambiar su dirección IP a menudo. Sin embargo, levanta la barrera de un ataque.

### 6.2 Secuestro de Cuenta

Muchas aplicaciones web hacen que sea fácil secuestrar cuentas de usuario. ¿Por qué no ser diferente y hacerlo más difícil ?.

#### 6.2.1 Contraseñas

Piense en una situación en la que un atacante ha robado la cookie de sesión de un usuario y, por lo tanto, puede usar la aplicación conjuntamente. Si es fácil cambiar la contraseña, el atacante secuestrará la cuenta con unos pocos clics. O si el formulario de cambio de contraseña es vulnerable a CSRF, el atacante podrá cambiar la contraseña de la víctima atrayéndolas a una página web donde hay una etiqueta IMG que hace la CSRF. Como contramedida, haga que los fomularios de cambio de contraseña sean seguras contra CSRF, por supuesto. Y requiera que el usuario introduzca la contraseña antigua al cambiarla.

#### 6.2.2 E-Mail

Sin embargo, el atacante también puede hacerse cargo de la cuenta cambiando la dirección de correo electrónico. Después de que lo cambien, irán a la página de contraseña olvidada y la \(posiblemente nueva\) contraseña será enviada a la dirección de correo electrónico del atacante. Como contramedida también debes requerir que el usuario introduzca la contraseña al cambiar la dirección de correo electrónico.

#### 6.2.3 Otros

Dependiendo de su aplicación web, puede haber más maneras de secuestrar la cuenta del usuario. En muchos casos CSRF y XSS ayudarán a hacerlo. Por ejemplo, como en una vulnerabilidad de CSRF en Gmail. En este ataque de prueba de concepto, la víctima habría sido atraída a un sitio web controlado por el atacante. En ese sitio hay una etiqueta IMG elaborada que da como resultado una solicitud HTTP GET que cambia la configuración del filtro de Google Mail. Si la víctima inició sesión en Google Mail, el atacante cambiaría los filtros para reenviar todos los correos electrónicos a su dirección de correo electrónico. Esto es casi tan dañino como secuestrar toda la cuenta. Como contramedida, revise la lógica de su aplicación y elimine todas las vulnerabilidades de XSS y CSRF.

### 6.3 CAPTCHAs

> Un CAPTCHA es una prueba de desafío-respuesta para determinar que la respuesta no es generada por una computadora. Se utiliza a menudo para proteger los formularios de registro de los atacantes y los formularios de comentarios de los robots de spam automáticos pidiendo al usuario que escriba las letras de una imagen distorsionada. Este es el CAPTCHA positivo, pero también hay el CAPTCHA negativo. La idea de un CAPTCHA negativo no es para un usuario probar que son humanos, pero revelan que un robot es un robot.

Una popular API CAPTCHA positiva es reCAPTCHA que muestra dos imágenes distorsionadas de palabras de libros antiguos. También añade una línea angulada, en lugar de un fondo distorsionado y altos niveles de deformación en el texto como anteriores CAPTCHAs, porque estos últimos se rompieron. Como bonificación, el uso de reCAPTCHA ayuda a digitalizar libros antiguos. [ReCAPTCHA](https://github.com/ambethia/recaptcha/) es también un complemento Rails con el mismo nombre que la API.

Obtendrá dos claves de la API, una clave pública y una privada, que debe poner en su entorno de Rails. Después de eso, puede utilizar el método `recaptcha_tags` en la vista y el método `verify_recaptcha` en el controlador. `verify_recaptcha` devolverá `false` si la validación falla. El problema con CAPTCHAs es que tienen un impacto negativo en la experiencia del usuario. Además, algunos usuarios con discapacidades visuales han encontrado ciertos tipos de CAPTCHA distorsionados difíciles de leer. Sin embargo, los CAPTCHA positivos son uno de los mejores métodos para evitar que todo tipo de bots envíen formularios.

La mayoría de los bots son realmente tontos. Ellos rastrean la web y ponen su spam en el campo de todos los formularios que pueden encontrar. Los CAPTCHA negativos se aprovechan de eso e incluyen un campo "`honeypot`" en el formulario que se oculta del usuario humano por CSS o JavaScript.

Tenga en cuenta que los CAPTCHA negativos sólo son eficaces contra bots mudos y no será suficiente para proteger las aplicaciones críticas de los bots seleccionados. Sin embargo, los CAPTCHA negativos y positivos pueden combinarse para aumentar el rendimiento, por ejemplo, si el campo "`honeypot`" no está vacío \(bot detectado\), no necesitará verificar el CAPTCHA positivo, lo que requeriría una solicitud HTTPS a Google ReCaptcha antes de calcular la respuesta.

Aquí están algunas ideas sobre cómo ocultar los campos del honeypot por JavaScript y/o CSS:

* Posicionar los campos fuera del área visible de la página
* Hacer los elementos muy pequeños o colorearlos igual que el fondo de la página
* Deje los campos mostrados, pero diga a los seres humanos que los dejen en blanco

El CAPTCHA negativo más simple es un campo escondido del honeypot. En el lado del servidor, comprobará el valor del campo: Si contiene cualquier texto, debe ser un bot. A continuación, puede ignorar la publicación o devolver un resultado positivo, pero no guardar la publicación en la base de datos. De esta manera el bot estará satisfecho y seguirá adelante. Usted puede hacer esto con usuarios molestos, también.

Usted puede encontrar CAPTCHA negativos más sofisticados en el [blog](https://nedbatchelder.com/text/stopbots.html) de Ned Batchelder:

* Incluya un campo con el sello de hora UTC actual y compruébelo en el servidor. Si está demasiado lejos en el pasado, o si es en el futuro, el formulario no es válido.
* Aleatorizar los nombres de los campos
* Incluir más de un campo de honeypot de todos los tipos, incluidos los botones de envío.

Tenga en cuenta que esto sólo lo protege de los bots automáticos, bots hechos a medida no puede ser detenido por esto. Así que los CAPTCHA negativos podrían no ser buenos para proteger los formularios de inicio de sesión.

### 6.4 Logging

> Dígale a Rails que no ponga contraseñas en los archivos de registro.

De forma predeterminada, Rails "loguea" \(hace logs\) todas las solicitudes que se realizan en la aplicación web. Pero los archivos de logs pueden ser un problema de seguridad enorme, ya que pueden contener credenciales de inicio de sesión, números de tarjetas de crédito y otros. Al diseñar un concepto de seguridad de aplicaciones web, también debe pensar en lo que sucederá si un atacante tiene acceso \(completo\) al servidor web. Cifrar los secrets y contraseñas en la base de datos será bastante inútil, si los archivos de logs los enuncian en texto claro. Puede filtrar ciertos parámetros de solicitud de sus archivos de logs agregándolos a `config.filter_parameters` en la configuración de la aplicación. Estos parámetros se marcarán \[FILTERED\] en el log.

```ruby
config.filter_parameters << :password
```

> Los parámetros proporcionados se filtrarán mediante una expresión regular coincidente parcial. Rails añade el valor por defecto `:password` en el inicializador apropiado \(`initialalizers/filter_parameter_logging.rb`\) y se preocupa por los parámetros típicos  `password` y `password_confirmation`de la aplicación

### 6.5 Buenas contraseñas

¿Le resulta difícil recordar todas sus contraseñas? No los escriba, sino que use las letras iniciales de cada palabra en una frase fácil de recordar.

Bruce Schneier, un tecnólogo de seguridad, [ha analizado](https://www.schneier.com/blog/archives/2006/12/realworld_passw.html) 34.000 nombres de usuario y contraseñas reales del ataque de phishing de MySpace que se menciona a continuación. Resulta que la mayoría de las contraseñas son bastante fáciles de romper. Las 20 contraseñas más comunes son:

password1, abc123, myspace1, password, blink182, qwerty1, \*\*\*\*you, 123abc, baseball1, football1, 123456, soccer, monkey1, liverpool1, princess1, jordan23, slipknot1, superman1, iloveyou1, and monkey.

Es interesante que sólo el 4% de estas contraseñas eran palabras del diccionario y la gran mayoría es en realidad alfanumérica. Sin embargo, los diccionarios de "crackeo" de contraseñas contienen un gran número de contraseñas de hoy, y prueban todo tipo de combinaciones \(alfanuméricas\). Si un atacante conoce su nombre de usuario y utiliza una contraseña débil, su cuenta se romperá fácilmente.

Una buena contraseña es una larga combinación alfanumérica de casos mixtos. Como esto es muy difícil de recordar, es aconsejable introducir sólo las primeras letras de una oración que pueda recordar fácilmente. Por ejemplo "El zorro marrón rápido salta sobre el perro perezoso" será "Tqbfjotld". Tenga en cuenta que esto es sólo un ejemplo, no debe utilizar frases bien conocidas como estas, ya que pueden aparecer en los diccionarios cracker, también.

### 6.6 Expresiones Regulares

> Una falla común en las expresiones regulares de Ruby es coincidir con la cadena que comienza y termina por ^ y $, en lugar de \A y \z.

Ruby utiliza un enfoque ligeramente diferente a muchos otros lenguajes para coincidir con el comienzo y el final de un string. Es por eso que incluso muchos libros de Ruby y Rails se equivocan. Entonces, ¿cómo es esto una amenaza de seguridad? Digamos que quería validar un campo de URL de manera flexible y utilizó una expresión regular simple como ésta:

```ruby
/^https?:\/\/[^\n]+$/i
```

Esto puede funcionar bien en algunos lenguajes. Sin embargo, en Ruby ^ y $ coinciden con el principio de la línea y el final de la línea. Y así una URL como ésta pasa el filtro sin problemas:

```js
javascript:exploit_code();/*
http://hi.com
*/
```



