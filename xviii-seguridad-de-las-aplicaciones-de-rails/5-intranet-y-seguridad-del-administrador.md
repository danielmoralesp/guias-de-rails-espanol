# 5- Intranet y seguridad del administrador

Las interfaces de la intranet y de la administración son blancos populares del ataque, porque permiten el acceso privilegiado. Aunque esto requeriría varias medidas de seguridad extra, lo contrario es el caso en el mundo real.

En 2007 se produjo el primer troyano hecho a la medida que robó información de una intranet, a saber, el sitio web Monster para empleadores de Monster.com, una aplicación web de reclutamiento en línea. Troyanos hechos a la medida son muy raros, hasta ahora, y el riesgo es bastante bajo, pero sin duda es una posibilidad y un ejemplo de cómo la seguridad del cliente de acogida es importante, también. Sin embargo, la mayor amenaza para las aplicaciones Intranet y Admin son _XSS_ y _CSRF_.

**XSS.** Si su aplicación vuelve a mostrar la entrada de usuario malintencionado desde la extranet, la aplicación será vulnerable a XSS. Los nombres de usuario, comentarios, informes de spam, direcciones de pedidos son sólo algunos ejemplos poco comunes, donde puede haber XSS.

Tener un solo lugar en la interfaz de administración o Intranet, donde la entrada no ha sido desinfectada, hace que toda la aplicación sea vulnerable. Las posibles ventajas incluyen robar la cookie del administrador privilegiado, inyectar un iframe para robar la contraseña del administrador o instalar software malicioso a través de los agujeros de seguridad del navegador para hacerse cargo del ordenador del administrador.

Consulte la sección de Inyección para las contramedidas en XSS.

**CSRF.** Cross-Site Request Forgery \(CSRF\), también conocido como Cross-Site Reference Forgery \(XSRF\), es un método de ataque gigantesco, que permite al atacante hacer todo lo que el administrador o usuario de Intranet puede hacer. Como ya has visto anteriormente, la CSRF funciona, aquí hay algunos ejemplos de lo que los atacantes pueden hacer en la Intranet o en la interfaz de administración.

Un ejemplo del mundo real es una reconfiguración del enrutador por CSRF. Los atacantes enviaron un correo electrónico malicioso, con CSRF en él, a los usuarios mexicanos. El e-mail afirmaba que había una tarjeta electrónica esperando al usuario, pero también contenía una etiqueta de imagen que resultó en una solicitud HTTP-GET para reconfigurar el enrutador del usuario \(que es un modelo popular en México\). La solicitud cambió los ajustes de DNS para que las solicitudes a un sitio bancario con sede en México se asignaran al sitio del atacante. Todos los que accedieron al sitio de la banca a través de ese enrutador vieron el sitio web falso del atacante y se les robaron sus credenciales.

Otro ejemplo cambió la dirección de correo electrónico y la contraseña de Google Adsense. Si la víctima ha iniciado sesión en Google Adsense, la interfaz de administración de campañas publicitarias de Google, un atacante podría cambiar las credenciales de la víctima.

Otro ataque popular es el spam de su aplicación web, su blog o foro para propagar XSS malicioso. Por supuesto, el atacante tiene que conocer la estructura de URL, pero la mayoría de las URLs de Rails son bastante sencillas o serán fáciles de averiguar, si se trata de una interfaz de administración de aplicación abierta. El atacante puede incluso hacer 1.000 suposiciones con suerte, incluyendo las etiquetas maliciosas IMG que prueban todas las posibles combinaciones.

Para las contramedidas contra CSRF en interfaces de administración e intranet, consulte las contramedidas en la sección CSRF.

### 5.1 Precauciones Adicionales

La interfaz de administración funciona de la siguiente manera: está ubicada en `www.example.com/admin`, sólo se puede acceder si el indicador de administrador se establece en el modelo `User`, vuelve a mostrar la entrada del usuario y permite al administrador borrar / agregar / editar lo que sea en los datos deseados. Aquí hay algunas reflexiones sobre esto:

* Es muy importante pensar en el peor caso: ¿Qué pasa si alguien realmente se apodera de sus cookies o credenciales de usuario. Podrías introducir funciones para la interfaz de administración para limitar las posibilidades del atacante. O qué tal las credenciales de inicio de sesión especiales para la interfaz de administración, que no sean las utilizadas para la parte pública de la aplicación. ¿O una contraseña especial para acciones muy graves?
* ¿El administrador realmente tiene que acceder a la interfaz desde cualquier parte del mundo? Piense en limitar el inicio de sesión a un montón de direcciones IP de origen. Examine `request.remote_ip` para obtener información sobre la dirección IP del usuario. Esto no es a prueba de balas, sino una gran barrera. Sin embargo, recuerde que puede haber un proxy en uso.
* Poner la interfaz de administración a un subdominio especial como `admin.application.com` y convertirlo en una aplicación independiente con su propia administración de usuarios. Esto hace que robar una cookie de administración del dominio habitual, `www.application.com`, sea imposible. Esto se debe a la misma política de origen en su navegador: Un script inyectado \(XSS\) en `www.application.com` no puede leer la cookie para `admin.application.com` y viceversa.













