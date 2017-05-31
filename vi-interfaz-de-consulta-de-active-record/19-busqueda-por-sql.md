# 19. Búsqueda por SQL

Si desea utilizar su propio `SQL` para buscar registros en una tabla, puede utilizar `find_by_sql`. El método `find_by_sql` devolverá una matriz de objetos incluso si la consulta subyacente devuelve un solo registro. Por ejemplo, puede ejecutar esta consulta:

```ruby
Client.find_by_sql("SELECT * FROM clients
  INNER JOIN orders ON clients.id = orders.client_id
  ORDER BY clients.created_at desc")
# =>  [
#   #<Client id: 1, first_name: "Lucas" >,
#   #<Client id: 2, first_name: "Jan" >,
#   ...
# ]
```

`find_by_sql` proporciona una forma sencilla de realizar llamadas personalizadas a la base de datos y recuperar objetos instanciados.

### 19.1 select\_all

`find_by_sql` tiene un pariente cercano llamado `connection#select_all`. `select_all` recuperará objetos de la base de datos usando `SQL` personalizado como `find_by_sql` pero no los instanciará. En su lugar, obtendrá una matriz de hashes donde cada hash indica un registro.

```ruby
Client.connection.select_all("SELECT first_name, created_at FROM clients WHERE id = '1'")
# => [
#   {"first_name"=>"Rafael", "created_at"=>"2012-11-10 23:23:45.281189"},
#   {"first_name"=>"Eileen", "created_at"=>"2013-12-09 11:22:35.221282"}
# ]
```

### 19.2 pluck

`pluck` se puede utilizar para consultar columnas múltiples o individuales de la tabla subyacente de un modelo. Acepta una lista de nombres de columna como argumento y devuelve una matriz de valores de las columnas especificadas con el tipo de datos correspondiente.

```ruby
Client.where(active: true).pluck(:id)
# SELECT id FROM clients WHERE active = 1
# => [1, 2, 3]

Client.distinct.pluck(:role)
# SELECT DISTINCT role FROM clients
# => ['admin', 'member', 'guest']

Client.pluck(:id, :name)
# SELECT clients.id, clients.name FROM clients
# => [[1, 'David'], [2, 'Jeremy'], [3, 'Jose']]
```

`pluck` hace posible reemplazar código como:

```ruby
Client.select(:id).map { |c| c.id }
# or
Client.select(:id).map(&:id)
# or
Client.select(:id, :name).map { |c| [c.id, c.name] }
```

por:

```ruby
Client.pluck(:id)
# or
Client.pluck(:id, :name)
```

A diferencia de `select`, `pluck` convierte directamente un resultado de base de datos en un Array de Ruby, sin construir objetos ActiveRecord. Esto puede significar un mejor rendimiento para una consulta grande o frecuente. Sin embargo, cualquier sobre-escritura del método del modelo no estará disponible. Por ejemplo:

```ruby
class Client < ApplicationRecord
  def name
    "I am #{super}"
  end
end

Client.select(:name).map &:name
# => ["I am David", "I am Jeremy", "I am Jose"]

Client.pluck(:name)
# => ["David", "Jeremy", "Jose"]
```

Además, a diferencia de `select` y otros ámbitos de Relación, `pluck` desencadena una consulta inmediata y, por lo tanto, no se puede encadenar con otros ámbitos, aunque puede trabajar con scopes ya construidos anteriormente:

```ruby
Client.pluck(:name).limit(1)
# => NoMethodError: undefined method `limit' for #<Array:0x007ff34d3ad6d8>

Client.limit(1).pluck(:name)
# => ["David"]
```

### 19.3 ids

`ids` se puede utilizar para extraer \(`pluck`\) todos los identificadores de la relación con la clave principal de la tabla.

```ruby
Person.ids
# SELECT id FROM people
```

```ruby
class Person < ApplicationRecord
  self.primary_key = "person_id"
end

Person.ids
# SELECT person_id FROM people
```



