# 1. Recuperación de objetos desde la base de datos

Para recuperar objetos de la base de datos, Active Record proporciona varios métodos `finder`. Cada método `finder` le permite pasar argumentos para realizar ciertas consultas en su base de datos sin escribir `SQL` sin procesar.

Los métodos son:

```ruby
find
create_with
distinct
eager_load
extending
from
group
having
includes
joins
left_outer_joins
limit
lock
none
offset
order
preload
readonly
references
reorder
reverse_order
select
where
```

Los métodos `finder` que devuelven una colección, como `where` y `group`, devuelven una instancia de `ActiveRecord::Relation`. Los métodos que encuentran una sola entidad, como `find` y `first`, devuelven una sola instancia del modelo.

La operación primaria de `Model.find(options)` se puede resumir como:

* Convierta las `options` suministradas en una consulta `SQL` equivalente. 
* Dispara la consulta `SQL` y recupera los resultados correspondientes de la base de datos. 
* Instancia el objeto Ruby equivalente del modelo apropiado para cada fila resultante. 
* Ejecute `after_find` y, a continuación, los callbacks de `after_initialize`, si las hubiere.

### 1.1 Recuperación de un único objeto

Active Record proporciona varias formas diferentes de recuperar un solo objeto.

#### 1.1.1 find

Utilizando el método `find`, puede recuperar el objeto correspondiente a la clave primaria especificada que coincida con las opciones suministradas. Por ejemplo:

```ruby
# Find the client with primary key (id) 10.
client = Client.find(10)
# => #<Client id: 10, first_name: "Ryan">
```

El equivalente `SQL` de lo anterior es:

```ruby
SELECT * FROM clients WHERE (clients.id = 10) LIMIT 1
```

El método `find` generará una excepción `ActiveRecord::RecordNotFound` si no se encuentra ningún registro coincidente.

También puede utilizar este método para consultar varios objetos. Llame al método `find` y pase en una matriz de claves primarias. La devolución será una matriz que contiene todos los registros coincidentes para las claves primarias suministradas. Por ejemplo:

```ruby
# Find the clients with primary keys 1 and 10.
client = Client.find([1, 10]) # Or even Client.find(1, 10)
# => [#<Client id: 1, first_name: "Lifo">, #<Client id: 10, first_name: "Ryan">]
```

El equivalente `SQL` de lo anterior es:

```ruby
SELECT * FROM clients WHERE (clients.id IN (1,10))
```

El método `find` generará una excepción `ActiveRecord::RecordNotFound` a menos que se encuentre un registro coincidente para todas las claves primarias suministradas.

#### 1.1.2 take

El método take recupera un registro sin ningún orden implícito. Por ejemplo:

```ruby
client = Client.take
# => #<Client id: 1, first_name: "Lifo">
```

El equivalente `SQL` de lo anterior es:

```ruby
SELECT * FROM clients LIMIT 1
```

El método `take` devuelve `nil` si no se encuentra ningún registro y no se generará ninguna excepción.

Puede pasar un argumento numérico al método `take` para devolver ese número de resultados. Por ejemplo

```ruby
client = Client.take(2)
# => [
#   #<Client id: 1, first_name: "Lifo">,
#   #<Client id: 220, first_name: "Sara">
# ]
```

El equivalente `SQL` de lo anterior es:

```ruby
SELECT * FROM clients LIMIT 2
```

El método `take!` se comporta exactamente como `take`, excepto que disparará `ActiveRecord::RecordNotFound` si no se encuentra ningún registro coincidente.

El registro recuperado puede variar dependiendo del motor de base de datos.

#### 1.1.3 first

El método `first` encuentra el primer registro ordenado por clave principal \(predeterminado\). Por ejemplo:

```ruby
client = Client.first
# => #<Client id: 1, first_name: "Lifo">
```

El equivalente `SQL` de lo anterior es:

```ruby
SELECT * FROM clients ORDER BY clients.id ASC LIMIT 1
```

El método `first` devuelve `nil` si no se encuentra ningún registro coincidente y no se genera ninguna excepción.

Si su scope predeterminado contiene un método `order`, `first` devolverá el primer registro de acuerdo con ese order.

Puede pasar un argumento numérico al primer método para traer ese número de resultados. Por ejemplo

```ruby
client = Client.first(3)
# => [
#   #<Client id: 1, first_name: "Lifo">,
#   #<Client id: 2, first_name: "Fifo">,
#   #<Client id: 3, first_name: "Filo">
# ]
```

El equivalente `SQL` de lo anterior es:

```ruby
SELECT * FROM clients ORDER BY clients.id ASC LIMIT 3
```

En una colección que se ordena utilizando `order`, `first` devolverá el primer registro ordenado por el atributo especificado para el `order`.

```ruby
client = Client.order(:first_name).first
# => #<Client id: 2, first_name: "Fifo">
```

El equivalente `SQL` de lo anterior es:

```ruby
SELECT * FROM clients ORDER BY clients.first_name ASC LIMIT 1
```

El método `first!` se comporta exactamente como `first`, excepto que disparará `ActiveRecord::RecordNotFound` si no se encuentra ningún registro coincidente.

#### 1.1.4 last

El método last encuentra el último registro ordenado por clave principal \(predeterminado\). Por ejemplo:

```ruby
client = Client.last
# => #<Client id: 221, first_name: "Russel">
```

El equivalente `SQL` de lo anterior es:

```ruby
SELECT * FROM clients ORDER BY clients.id DESC LIMIT 1
```

El método `last` devuelve `nil` si no se encuentra ningún registro coincidente y no se generará ninguna excepción.

Si su scope predeterminado contiene un método `order`, `last` devolverá el último registro de acuerdo con este `order`.

Puede pasar un argumento numérico al método last para traer ese número de resultados. Por ejemplo

```ruby
client = Client.last(3)
# => [
#   #<Client id: 219, first_name: "James">,
#   #<Client id: 220, first_name: "Sara">,
#   #<Client id: 221, first_name: "Russel">
# ]
```

El equivalente `SQL` de lo anterior es:

```ruby
SELECT * FROM clients ORDER BY clients.id DESC LIMIT 3
```

En una colección que se ordena utilizando `order`, `last` devolverá el último registro ordenado por el atributo especificado para `order`.

```ruby
client = Client.order(:first_name).last
# => #<Client id: 220, first_name: "Sara">
```

El equivalente `SQL` de lo anterior es:

```ruby
SELECT * FROM clients ORDER BY clients.first_name DESC LIMIT 1
```

El método `last!` se comporta exactamente como `last`, excepto que disparará `ActiveRecord::RecordNotFound` si no se encuentra ningún registro coincidente.

#### 1.1.5 find\_by

El método `find_by` encuentra el primer registro que coincide con algunas condiciones. Por ejemplo:

```ruby
Client.find_by first_name: 'Lifo'
# => #<Client id: 1, first_name: "Lifo">

Client.find_by first_name: 'Jon'
# => nil
```

Es equivalente a escribir

```ruby
Client.where(first_name: 'Lifo').take
```

El equivalente `SQL` de lo anterior es:

```ruby
SELECT * FROM clients WHERE (clients.first_name = 'Lifo') LIMIT 1
```

El método `find_by!`se comporta exactamente como `find_by`, excepto que disparará `ActiveRecord::RecordNotFound` si no se encuentra ningún registro coincidente. Por ejemplo:

```ruby
Client.find_by! first_name: 'does not exist'
# => ActiveRecord::RecordNotFound
```

Esto es equivalente a escribir

```ruby
Client.where(first_name: 'does not exist').take!
```

### 1.2 Recuperación de varios objetos en lotes

A menudo necesitamos iterar sobre un gran conjunto de registros, como cuando enviamos un boletín a un gran grupo de usuarios o cuando exportamos datos.

Esto puede parecer sencillo:

```ruby
# This may consume too much memory if the table is big.
User.all.each do |user|
  NewsMailer.weekly(user).deliver_now
end
```

Pero este enfoque se vuelve poco práctico a medida que aumenta el tamaño de la tabla, ya que `User.all.each` le indica a Active Record que busque toda la tabla en un solo paso, construya un objeto de modelo por fila y guarde toda la matriz de objetos de modelo en memoria. De hecho, si tenemos un gran número de registros, toda la colección puede exceder la cantidad de memoria disponible.

Rails proporciona dos métodos que solucionan este problema dividiendo los registros en lotes compatibles con la memoria para su procesamiento. El primer método, `find_each`, recupera un lote de registros y luego produce cada registro en el bloque individualmente como un modelo. El segundo método, `find_in_batches`, recupera un lote de registros y luego produce el lote completo en el bloque como una matriz de modelos.

> Los métodos `find_each` y `find_in_batches` están diseñados para su uso en el procesamiento por lotes de un gran número de registros que no caben en la memoria de una sola vez. Si sólo necesita ejecutar la búsqueda en alrededor de mil registros, los métodos de búsqueda regulares son la opción preferida.

#### 1.2.1`find_each`

El método `find_each` recupera los registros en lotes y luego reproduce cada uno en el bloque. En el ejemplo siguiente, `find_each` recupera usuarios en lotes de 1000 y los entrega al bloque uno a uno:

```ruby
User.find_each do |user|
  NewsMailer.weekly(user).deliver_now
end
```

Este proceso se repite, obteniendo más lotes según sea necesario, hasta que todos los registros hayan sido procesados.

`find_each` trabaja sobre las clases de los modelos, como se ha visto anteriormente, y también sobre las relaciones:

```ruby
User.where(weekly_subscriber: true).find_each do |user|
  NewsMailer.weekly(user).deliver_now
end
```

Siempre y cuando no tengan un `order`, ya que el método necesita forzar un orden interno para iterar.

Si un método `order` está presente en el receptor, el comportamiento depende del indicador `config.active_record.error_on_ignored_order`. Si es `true`, `ArgumentError` se dispara, de lo contrario se ignora el orden y se emite una advertencia, que es la predeterminada. Esto puede anularse con la opción: `error_on_ignore`, explicada abajo.

##### 1.2.1.1 Opciones para`find_each`

**`:batch_size`**

La opción: `batch_size` le permite especificar el número de registros que se recuperarán en cada lote, antes de pasar individualmente al bloque. Por ejemplo, para recuperar registros en lotes de 5000:

```ruby
User.find_each(batch_size: 5000) do |user|
  NewsMailer.weekly(user).deliver_now
end
```

**`:start`**

De forma predeterminada, los registros se obtienen en orden ascendente de la clave principal, que debe ser un entero. La opción: `start` le permite configurar el primer `ID` de la secuencia siempre que el `ID` más bajo no sea el que necesita. Esto sería útil, por ejemplo, si desea reanudar un proceso por lotes interrumpido, siempre que haya guardado el último `ID` procesado como punto de control.

Por ejemplo, para enviar boletines sólo a usuarios con la clave principal a partir de 2000:

```ruby
User.find_each(start: 2000) do |user|
  NewsMailer.weekly(user).deliver_now
end
```

**`:finish`**

Similar a la opción: `start`,: `finish` le permite configurar el último `ID` de la secuencia siempre que el `ID` más alto no sea el que usted necesita. Esto sería útil, por ejemplo, si desea ejecutar un proceso por lotes utilizando un subconjunto de registros basado en: `start` y: `finish`.

Por ejemplo, para enviar boletines sólo a usuarios con la clave principal desde 2000 hasta 10000:

```ruby
User.find_each(start: 2000, finish: 10000) do |user|
  NewsMailer.weekly(user).deliver_now
end
```

Otro ejemplo sería si desea que varios workers manejen la misma cola de procesamiento. Podría hacer que cada worker maneje 10000 registros estableciendo las opciones apropiadas: `start` y: `finish` en cada worker.

**`:error_on_ignore`**

Anula la configuración de la aplicación para especificar si se debe generar un error cuando hay un pedido en la relación.

#### 1.2.2`find_in_batches`

El método `find_in_batches` es similar a `find_each`, ya que ambos recuperan lotes de registros. La diferencia es que `find_in_batches` produce lotes al bloque como un array de modelos, en lugar de individualmente. El ejemplo siguiente cederá al bloque suministrado una matriz de hasta 1000 facturas a la vez, con el bloque final que contiene las facturas restantes:

```ruby
# Give add_invoices an array of 1000 invoices at a time.
Invoice.find_in_batches do |invoices|
  export.add_invoices(invoices)
end
```

`find_in_batches` trabaja en las clases de los modelos, como se ha visto anteriormente, y también en las relaciones:

```ruby
Invoice.pending.find_in_batches do |invoice|
  pending_invoices_export.add_invoices(invoices)
end
```

Siempre y cuando no tengan un `order`, ya que el método necesita forzar un orden interno para iterar.

##### 1.2.2.1 Opciones para `find_in_batches`

El método `find_in_batches` acepta las mismas opciones que `find_each`.



