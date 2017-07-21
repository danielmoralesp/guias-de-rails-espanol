# 7- Injection

> Injection es una clase de ataques que introducen código malicioso o parámetros en una aplicación web para ejecutarlo dentro de su contexto de seguridad. Los ejemplos prominentes de la inyección son secuencias de comandos entre sitios \(XSS\) e inyección de SQL.

La inyección es muy complicada, porque el mismo código o parámetro puede ser malicioso en un contexto, pero totalmente inofensivo en otro. Un contexto puede ser un lenguaje de programación, consulta o de programación, el shell o un método Ruby / Rails. Las secciones siguientes cubrirán todos los contextos importantes donde pueden ocurrir ataques de inyección. La primera sección, sin embargo, cubre una decisión arquitectónica en relación con la inyección.

### 7.1 Listas blancas versus listas negras

> Al desinfectar, proteger o verificar algo, prefiera las listas blancas sobre listas negras.

Una lista negra puede ser una lista de direcciones de correo electrónico incorrectas, acciones no públicas o etiquetas HTML incorrectas. Esto se opone a una lista blanca que enumera las buenas direcciones de correo electrónico, acciones públicas, buenas etiquetas HTML y así sucesivamente. Aunque a veces no es posible crear una lista blanca \(en un filtro SPAM, por ejemplo\), prefiera utilizar enfoques de listas blancas:

* Utilice `before_action except: [...] instead of only: [...]`  para acciones relacionadas con la seguridad. De esta manera, no olvide activar las comprobaciones de seguridad de las acciones añadidas recientemente.

* Permitir &lt;strong&gt; en lugar de eliminar &lt;script&gt; contra Cross-Site Scripting \(XSS\). Vea a abajo para más detalles

* No trate de corregir la entrada de usuarios mediante listas negras:

  * Esto hará que el ataque funcione: `"<sc<script>ript>".gsub("<script>", "")`

  * Pero rechazar el input malformada

Las listas blancas son también un buen enfoque contra el factor humano de olvidar algo en la lista negra.

### 7.2 Inyección de SQL

Gracias a los métodos inteligentes, esto no es un problema en la mayoría de las aplicaciones de Rails. Sin embargo, este es un ataque muy devastador y común en las aplicaciones web, por lo que es importante entender el problema.

#### 7.2.1 Introducción

Los ataques de inyección SQL apuntan a influenciar las consultas de la base de datos mediante la manipulación de los parámetros de la aplicación web. Un objetivo popular de los ataques de inyección de SQL es evitar la autorización. Otro objetivo es realizar la manipulación de datos o leer datos arbitrarios. A continuación se muestra un ejemplo de cómo no utilizar datos de entrada del usuario en una consulta:

```ruby
Project.where("name = '#{params[:name]}'")
```

Esto podría estar en un action llamado `search` y el usuario puede introducir el nombre de un proyecto que desea encontrar. Si un usuario malintencionado introduce `'OR 1 --`, la consulta SQL resultante será:

```ruby
SELECT * FROM projects WHERE name = '' OR 1 --'
```

Los dos guiones empiezan un comentario ignorando todo después de él. Por lo tanto, la consulta devuelve todos los registros de la tabla de proyectos, incluidos los ocultos para el usuario. Esto se debe a que la condición es `true`para todos los registros.

#### 7.2.2 Omisión de la autorización

Por lo general, una aplicación web incluye control de acceso. El usuario introduce sus credenciales de inicio de sesión y la aplicación web trata de encontrar el registro coincidente en la tabla de usuarios. La aplicación concede acceso cuando encuentra un registro. Sin embargo, un atacante puede evitar esta comprobación con la inyección de SQL. A continuación se muestra una consulta de base de datos típica en Rails para encontrar el primer registro en la tabla de usuarios que coincide con los parámetros de credenciales de inicio de sesión suministrados por el usuario.

```ruby
User.find_by("login = '#{params[:name]}' AND password = '#{params[:password]}'")
```

Si un atacante introduce ' OR '1'='1 como nombre y ' OR '2'&gt;'1 como contraseña, la consulta SQL resultante será:

```ruby
SELECT * FROM users WHERE login = '' OR '1'='1' AND password = '' OR '2'>'1' LIMIT 1
```

Esto simplemente encontrará el primer registro en la base de datos y concede acceso a este usuario.

#### 7.2.3 Lectura no autorizada

La instrucción UNION conecta dos consultas SQL y devuelve los datos en un conjunto. Un atacante puede usarlo para leer datos arbitrarios de la base de datos. Tomemos el ejemplo de arriba:

```ruby
Project.where("name = '#{params[:name]}'")
```

Y ahora vamos a inyectar otra consulta utilizando la sentencia UNION:

```ruby
') UNION SELECT id,login AS name,password AS description,1,1,1 FROM users --
```

Esto resultará en la siguiente consulta SQL:

```ruby
SELECT * FROM projects WHERE (name = '') UNION
  SELECT id,login AS name,password AS description,1,1,1 FROM users --'
```

El resultado no será una lista de proyectos \(porque no hay un proyecto con un nombre vacío\), sino una lista de nombres de usuario y su contraseña. Así que esperamos que haya encriptado las contraseñas en la base de datos! El único problema para el atacante es que el número de columnas tiene que ser el mismo en ambas consultas. Es por eso que la segunda consulta incluye una lista de unos \(1\), que será siempre el valor 1, con el fin de coincidir con el número de columnas en la primera consulta.

Además, la segunda consulta cambia el nombre de algunas columnas con la instrucción AS para que la aplicación web muestre los valores de la tabla de usuario. Asegúrese de actualizar sus Rails a al menos 2.1.1.

#### 7.2.4 Contramedidas

Ruby on Rails tiene un filtro incorporado para caracteres especiales de SQL, que escapará ",", caracteres NULL y saltos de línea.Utilizando `Model.find(id)` o `Model.find_by_something(something)` aplica automáticamente esta contramedida. Para fragmentos, especialmente en condiciones de fragmentos \(`where("...")`\), los métodos `connection.execute()`o `Model.find_by_sql()`, tiene que aplicarse manualmente.

En lugar de pasar una cadena a la opción de condiciones, puede pasar una matriz para desinfectar cadenas viciadas así:

```ruby
Model.where("login = ? AND password = ?", entered_user_name, entered_password).first
```

Como puede ver, la primera parte de la matriz es un fragmento de SQL con signos de interrogación. Las versiones desinfectadas de las variables de la segunda parte de la matriz sustituyen a los signos de interrogación. O puede pasar un hash para el mismo resultado:

```ruby
Model.where(login: entered_user_name, password: entered_password).first
```

La matriz o hash de los formularios sólo está disponible en instancias de modelo. Puedes intentar el `sanitize_sql()`en otro lugar. Haga un hábito pensar en las consecuencias de seguridad cuando se utiliza una cadena externa en SQL.

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



