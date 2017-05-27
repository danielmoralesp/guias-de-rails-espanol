# VI- Interfaz de Consulta de Active Record

Esta guía cubre diferentes maneras de recuperar datos de la base de datos con Active Record.

Después de leer esta guía, sabrá:

* Cómo encontrar registros usando una variedad de métodos y condiciones. 
* Cómo especificar el orden, los atributos recuperados, el agrupamiento y otras propiedades de los registros encontrados. 
* Cómo utilizar la carga ansiosa para reducir el número de consultas de base de datos necesarias para la recuperación de datos. 
* Cómo utilizar los métodos de búsqueda dinámica. 
* Cómo utilizar el método de encadenamiento para utilizar múltiples métodos Active Record juntos. 
* Cómo comprobar la existencia de registros particulares. 
* Cómo realizar varios cálculos en los modelos Active Record. 
* Cómo ejecutar `EXPLAIN` en las relaciones.

Si está acostumbrado a usar `SQL` sin procesar para buscar registros de base de datos, generalmente encontrará que hay mejores maneras de llevar a cabo las mismas operaciones en Rails. Active Record le aísla de la necesidad de usar `SQL` en la mayoría de los casos.

Los ejemplos de código a lo largo de esta guía se referirán a uno o más de los siguientes modelos:

Todos los modelos siguientes usan `id` como clave principal, a menos que se especifique lo contrario.

```ruby
class Client < ApplicationRecord
  has_one :address
  has_many :orders
  has_and_belongs_to_many :roles
end

class Address < ApplicationRecord
  belongs_to :client
end

class Order < ApplicationRecord
  belongs_to :client, counter_cache: true
end

class Role < ApplicationRecord
  has_and_belongs_to_many :clients
end
```

Active Record realizará consultas en la base de datos para usted y es compatible con la mayoría de los sistemas de bases de datos, incluyendo MySQL, MariaDB, PostgreSQL y SQLite. Independientemente del sistema de base de datos que esté utilizando, el formato del método Active Record siempre será el mismo.

