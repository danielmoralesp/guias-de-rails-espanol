# 2- Sesiones

Un buen lugar para empezar a mirar la seguridad es en las sesiones, que pueden ser vulnerables a los ataques en particular.

### 2.1 ¿Qué son las sesiones?

> HTTP es un protocolo sin estado. Las sesiones lo hacen con estado.

La mayoría de las aplicaciones necesitan seguir un determinado estado de un usuario en particular. Éste podría ser el contenido de una cesta de la compra o el ID de usuario que actualmente está conectado. Sin la idea de las sesiones, el usuario tendría que identificarse, y probablemente autenticarse, en cada solicitud. Rails creará una nueva sesión automáticamente si un nuevo usuario accede a la aplicación. Se cargará una sesión existente si el usuario ya ha utilizado la aplicación.

Una sesión normalmente consiste en un hash de valores y un ID de sesión, usualmente una cadena de 32 caracteres, para identificar el hash. Cada cookie enviada al navegador del cliente incluye el ID de sesión. Y al revés: el navegador lo enviará al servidor en cada solicitud del cliente. En Rails puede guardar y recuperar valores mediante el método de sesión:

```ruby
session[:user_id] = @current_user.id
User.find(session[:user_id])
```

### 2.2 ID de sesión

> El identificador de sesión es una cadena hexadecimal aleatoria de 32 caracteres.

El identificador de sesión se genera utilizando `SecureRandom.hex` que genera una cadena hexadecimal aleatoria utilizando métodos específicos de plataforma \(como OpenSSL, `/dev/urandom` o Win32\) para generar números aleatorios criptográficamente seguros. Actualmente no es factible para los IDs de sesión de Rails de fuerza bruta.

### 2.3 Secuestro de sesión

> Robar el ID de sesión de un usuario permite que un atacante use la aplicación web en nombre de la víctima.

Muchas aplicaciones web tienen un sistema de autenticación: un usuario proporciona un nombre de usuario y una contraseña, la aplicación web los comprueba y almacena el identificador de usuario correspondiente en el hash de sesión. A partir de ahora, la sesión es válida. En cada solicitud la aplicación cargará al usuario, identificado por el identificador de usuario en la sesión, sin necesidad de una nueva autenticación. El ID de sesión en la cookie identifica la sesión.

Por lo tanto, la cookie sirve como autenticación temporal para la aplicación web. Cualquier persona que se apodera de una cookie de otra persona, puede utilizar la aplicación web como este usuario - con posibles consecuencias graves. Aquí hay algunas maneras de secuestrar una sesión, y sus contramedidas:

* Olfatea la cookie en una red insegura. Una LAN inalámbrica puede ser un ejemplo de tal red. En una LAN inalámbrica sin cifrar, es especialmente fácil escuchar el tráfico de todos los clientes conectados. Para el generador de aplicaciones web, esto significa proporcionar una conexión segura a través de SSL. En Rails 3.1 y posteriores, esto podría lograrse forzando siempre la conexión SSL en el archivo de configuración de la aplicación:

```ruby
config.force_ssl = true
```

* La mayoría de la gente no limpia las cookies después de trabajar en una terminal pública. Por lo tanto, si el último usuario no se desconectó de una aplicación web, podría utilizarla como usuario. Proporcione al usuario un botón de cierre de sesión en la aplicación web y haga que se destaque.
* Muchos intentos de scripts entre sitios \(XSS\) buscan obtener la cookie del usuario. Más adelante leerá más sobre XSS.
* En lugar de robar una cookie desconocida por el atacante, ellos corrigen el identificador de sesión de un usuario \(en la cookie\) que conocen. Lea más sobre esta sesión de fijación más adelante.

El objetivo principal de la mayoría de los atacantes es ganar dinero. Los precios en el bajo mundo para las cuentas robadas de acceso a bancos van de $10 - $1000 \(dependiendo de la cantidad disponible de fondos\), de $0.40- $20 para números de tarjeta de crédito, $1- $8 para las cuentas en línea de un sitio de subasta y $4- $30 para las contraseñas de email, según el Informe de amenazas de [Symantec Global Internet Security](http://eval.symantec.com/mktginfo/enterprise/white_papers/b-whitepaper_internet_security_threat_report_xiii_04-2008.en-us.pdf).

### 2.4 Directrices de la Sesión

Aquí hay algunas pautas generales sobre las sesiones.

* No guarde objetos grandes en una sesión. En su lugar debe guardarlos en la base de datos y guardar su id en la sesión. Esto eliminará los dolores de cabeza de sincronización y no llenará el espacio de almacenamiento de sesión \(dependiendo del almacenamiento de sesiones que elija, consúltelo más adelante\). Esto también será una buena idea, si modifica la estructura de un objeto y las versiones antiguas de él todavía están en las cookies de algunos usuarios. Con los almacenes de sesión del lado del servidor puede borrar las sesiones, pero con los almacenes del lado del cliente, esto es difícil de mitigar.
* Los datos críticos no deben almacenarse en una sesión. Si el usuario borra sus cookies o cierra el navegador, se perderán. Y con un almacenamiento de sesión del lado del cliente, el usuario puede leer los datos.

### 2.5 Almacenamiento de sesión

> Rails proporciona varios mecanismos de almacenamiento para los hashes de sesión. El más importante es `ActionDispatch::Session::CookieStore`.

Rails 2 introdujo un nuevo almacenamiento de sesión predeterminado, CookieStore. CookieStore guarda el hash de sesión directamente en una cookie en el lado del cliente. El servidor recupera el hash de sesión de la cookie y elimina la necesidad de un ID de sesión. Eso aumentará en gran medida la velocidad de la aplicación, pero es una opción de almacenamiento controvertida y hay que pensar en las implicaciones de seguridad de la misma:

* Las cookies implican un límite de tamaño estricto de 4kB. Esto está bien, ya que no debe almacenar grandes cantidades de datos en una sesión de todos modos, como se describió anteriormente. Almacenar el ID de la base de datos del usuario actual en una sesión es normalmente aceptable.
* El cliente puede ver todo lo que almacena en una sesión, ya que se almacena en texto claro \(en realidad codificado en Base64, no cifrado\). Así que, por supuesto, usted no quiere guardar ningún secreto aquí. Para evitar la manipulación de hash de sesión, se calcula un digest a partir de la sesión con un secreto de servidor \(secrets.secret\_token\) e insertado en el final de la cookie.

Sin embargo, desde Rails 4, la tienda predeterminada es `EncryptedCookieStore`. Con `EncryptedCookieStore` la sesión se cifra antes de ser almacenada en una cookie. Esto evita que el usuario acceda y altere el contenido de la cookie. Por lo tanto, la sesión se convierte en un lugar más seguro para almacenar datos. El cifrado se realiza mediante una clave secreta del servidor `secrets.secret_key_base` almacenada en `config/secrets.yml`.

Esto significa que la seguridad de este almacenamiento depende de este secret \(y en el algoritmo digest, que por defecto a SHA1, para la compatibilidad\). Por lo tanto, no utilice un secret trivial, es decir, una palabra de un diccionario, o una que sea más corta de 30 caracteres, use los rails secret en su lugar.

`secrets.secret_key_base` se utiliza para especificar una clave que permite que las sesiones de la aplicación se verifiquen frente a una clave segura conocida para evitar la manipulación indebida. Las aplicaciones obtienen `secrets.secret_key_base` inicializado a una clave aleatoria presente en `config/secrets.yml`, por ejemplo:

```ruby
development:
  secret_key_base: a75d...
 
test:
  secret_key_base: 492f...
 
production:
  secret_key_base: <%= ENV["SECRET_KEY_BASE"] %>
```

Las versiones anteriores de Rails utilizan CookieStore, que utiliza `secret_token` en lugar de `secret_key_base` que es utilizado por `EncryptedCookieStore`. Lea la documentación de actualización para obtener más información.

Si ha recibido una aplicación en la que se descubrió el secret \(por ejemplo, una aplicación cuyo código fuente se compartió\), considere seriamente cambiar el secret.

### 2.6 Ataques de repetición para las sesiones de CookieStore

> Otro tipo de ataque que debes tener en cuenta al usar CookieStore es el ataque de repetición.

Funciona así:

* Un usuario recibe créditos, la cantidad se almacena en una sesión \(que es una mala idea de todos modos, pero lo haremos para propósitos de demostración\).
* El usuario compra algo.
* El nuevo valor de crédito ajustado se almacena en la sesión.
* El usuario toma la cookie del primer paso \(que previamente copió\) y reemplaza la cookie actual en el navegador.
* El usuario tiene su crédito original de nuevo.

Incluir un nonce \(un valor aleatorio\) en la sesión resuelve ataques de repetición. Un nonce es válido sólo una vez, y el servidor tiene que hacer un seguimiento de todos los nonces válidos. Se vuelve aún más complicado si tiene varios servidores de aplicaciones. Almacenar nonces en una tabla de la base de datos anularía el propósito entero de `CookieStore` \(evitar el acceso a la base de datos\).

La mejor solución en contra es no almacenar este tipo de datos en una sesión, sino en la base de datos. En este caso, guarde el crédito en la base de datos y en el `session_in_user_id` en la sesión.

### 2.7 Fijación de sesión

> Aparte de robar el ID de sesión de un usuario, el atacante puede arreglar un ID de sesión conocido por ellos. Esto se llama fijación de sesión.

![](/assets/Captura de pantalla 2017-07-20 a las 9.15.47 a.m..png)



Este ataque se centra en la fijación de un ID de sesión del usuario conocido por el atacante, y obligando al navegador del usuario a utilizar este ID. Por lo tanto, no es necesario que el atacante robe el ID de sesión después. Así es como funciona este ataque:

* El atacante crea un ID de sesión válido: Cargan la página de inicio de sesión de la aplicación web en la que quieren arreglar la sesión y toman el ID de sesión en la cookie de la respuesta \(consulte los números 1 y 2 de la imagen\).
* Ellos mantienen la sesión accediendo a la aplicación web periódicamente con el fin de mantener viva una sesión que expira.
* El atacante obliga al navegador del usuario a usar este ID de sesión \(ver número 3 en la imagen\). Como no puede cambiar una cookie de otro dominio \(debido a la misma política de origen\), el atacante debe ejecutar JavaScript desde el dominio de la aplicación web de destino. Inyectar el código JavaScript en la aplicación por XSS logra este ataque. Aquí está un ejemplo:
  `<script> document.cookie = "_ session_id = 16d5b78abb28e3d6206b60f22a03c8d9"; </ script>`   Más información sobre XSS e inyección más adelante.

* El atacante atrae a la víctima a la página infectada con el código JavaScript. Al ver la página, el navegador de la víctima cambiará el ID de sesión al ID de sesión de captura.
* Como la nueva sesión de captura no se utiliza, la aplicación web requerirá que el usuario se autentique.
* A partir de ahora, la víctima y el atacante usarán conjuntamente la aplicación web con la misma sesión: La sesión se hizo válida y la víctima no se dio cuenta del ataque.

### 2.8 Fijación de sesión - Contramedidas

> Una línea de código le protegerá de la fijación de la sesión.

La contramedida más efectiva es emitir un nuevo identificador de sesión y declarar el antiguo inválido después de un inicio de sesión satisfactorio. De esta manera, un atacante no puede usar el identificador de sesión fijo. Esta es una buena contramedida contra el secuestro de sesión. A continuación se muestra cómo crear una nueva sesión en Rails:

```ruby
reset_session
```

Si utiliza la popular gema Devise para la gestión de usuarios, ésta expirará automáticamente las sesiones de inicio y cierre de sesión por usted. Si hace la suya, recuerde expirar la sesión después de la acción de inicio de sesión \(cuando se cree la sesión\). Esto eliminará los valores de la sesión, por lo que tendrá que transferirlos a la nueva sesión.

Otra contramedida es guardar las propiedades específicas del usuario en la sesión, verificarlas cada vez que una solicitud llegue y denegar el acceso, si la información no coincide. Tales propiedades podrían ser la dirección IP remota o el agente de usuario \(el nombre del navegador web\), aunque este último es menos específico que la del usuario. Al guardar la dirección IP, hay que tener en cuenta que hay proveedores de servicios de Internet o grandes organizaciones que colocan a sus usuarios detrás de los proxies. Estos cambios pueden cambiar durante el curso de una sesión, por lo que estos usuarios no podrán utilizar su aplicación, o sólo de forma limitada.

### 2.9 Expiración de la sesión

> Las sesiones que nunca caducan prolongan el marco de tiempo para ataques como la falsificación de solicitudes entre sitios \(CSRF\), el secuestro de sesión y la fijación de sesión.

Una posibilidad es establecer la marca de tiempo de caducidad de la cookie con el ID de sesión. Sin embargo, el cliente puede editar las cookies que se almacenan en el navegador web para expirar las sesiones en el servidor, haciéndolo así más seguro. Este es un ejemplo de cómo expirar las sesiones en una tabla de base de datos. Llame a `Session.sweep("20 minutes")` para que caduquen las sesiones que se utilizaron hace más de 20 minutos.

```ruby
class Session < ApplicationRecord
  def self.sweep(time = 1.hour)
    if time.is_a?(String)
      time = time.split.inject { |count, unit| count.to_i.send(unit) }
    end
 
    delete_all "updated_at < '#{time.ago.to_s(:db)}'"
  end
end
```

La sección sobre "la fijación de la sesión" introdujo el problema de las sesiones mantenidas. Un atacante que mantiene una sesión cada cinco minutos puede mantener la sesión viva para siempre, aunque esté expirando las sesiones. Una solución simple para esto sería agregar una columna `created_at` a la tabla de sesiones. Ahora puede eliminar las sesiones que se crearon hace mucho tiempo. Utilice esta línea en el método de barrido anterior:

```ruby
delete_all "updated_at < '#{time.ago.to_s(:db)}' OR
  created_at < '#{2.days.ago.to_s(:db)}'"
```























