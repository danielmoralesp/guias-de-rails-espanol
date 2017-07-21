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



