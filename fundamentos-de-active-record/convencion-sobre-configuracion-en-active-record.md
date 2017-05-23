# 2. Convención sobre Configuración en Active Record

Cuando escribimos aplicaciones usando otros lenguajes de programación o frameworks, es necesario escribir una gran cantidad de código de configuración. Esto es particularmente verdad para los frameworks ORM en general. Sin embargo, si sigues las convenciones adoptadas por Rails, necesitarás escribir muy poca configuración \(en algunos casos ninguna\) cuando creas modelos Active Record. La idea es que si configuras tus aplicaciones de la misma manera la mayoría de las veces, entonces ese debería ser la manera por defecto. Por consiguiente, podríamos necesitar configuración explícita solo en los casos donde no podemos seguir la convención standart.



### 2.1 Convenciones sobre Nombres 

Por defecto, Active Record utiliza algunas convenciones para encontrar y conocer con detalle el mapeo entre los modelos y las tablas de la base de datos y como debería crearse. Rails convertirá al plural los nombres de tus clases para encontrar la respectiva tabla en la base de datos. Entonces, para una clase `Book`, deberías tener una tabla de base de datos llamada `books.`Los mecanismos de pluralizar en Rails son muy potentes, y tienen la capacidad de pluralizarar \(y singularizar\) ambos en palabras regulares e irregulares. Cuando usamos nombres de clases compuestos de dos o más palabras, el nombre de la clase del modelo debe debería seguir las convenciones Ruby, usando la forma CamelCase, mientras que el nombre de la tabla debe contener las palabras separadas por guiones bajos. Ejemplos:

* Tabla de Base de Datos - Plural con guiones bajos separando las palabras \(ej: `book_clubs`\). 
* Clase del Modelo - Singular con la primera letra de cada palabra en mayúsculas \(ej: `BookClub`\).



| Modelo / Clase | Tabla / Esquema |
| :--- | :--- |
| Article | articles |
| LineItem | line\_items |
| Deer | deers |
| Mouse | mice |
| Person | people |



### 2.2 Convenciones del Esquema

Active Record utiliza convenciones de nombres para las columnas en las tablas de la base de datos, dependiendo de los propósitos de esas columnas. 

* Claves foráneas - Estos campos deberían ser nombrados siguiendo el patrón nombre\_de\_tabla\_en\_singular\_id \(ej: `item_id`, `order_id`\). Estos son los campos que Active Record buscará cuando crees asociaciones entre tus modelos. 
* Claves primarias - Por defecto, Active Record utilizará una columna entera llamada id como la clave primaria de la tabla. Cuando utilizas Active Record Migrations para crear tus tablas, esta columna será creada automáticamente.

También hay nombres de columnas opcionales que podrán dar características adicionales a las instancias de Active Record: 

* `created_at` - Automáticamente guarda el dia y la hora actual al momento en que se crea el objeto. 
* `updated_at` - Automáticamente guarda el día y la hora actual al momento en que se actualiza un registro. 
* `lock_version` - Añade bloqueo optimista a un modelo. 
* `type` - Especifica el modo de utilizar el modelo La Herencia Simple de Tables \(Simple Table Inheritance - STI\)
* `(association_name)_type` - Graba el tipo para asociaciones polimórficas. 
* `(table_name)_count` - Utilizado para cachear el número de objetos que pertenecientes en una asociación. 



Por ejemplo, una columna `comments_count` en una clase `Articles` que tiene muchas instancias de `Comment` guardará en cache el número de los comentarios existentes de cada artículo.

Mientras estos nombres de columnas son opcionales, están en realidad reservados por Active Record. Evita el uso de las palabras reservadas si no quieres funcionalidades extra. Por ejemplo, `type` es una palabra reservada utilizada para definir que una tabla está utilizando Herencia Simple de Tabla \(STI\). Si no estás utilizando STI, intenta con una palabra análoga como `context`, que puede aún así mantener la descripción de los datos que estás modelando.



