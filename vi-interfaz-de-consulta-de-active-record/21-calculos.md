# 21. Cálculos

Esta sección usa `count` como un método de ejemplo en este preámbulo, pero las opciones descritas se aplican a todas las subsecciones.

Todos los métodos de cálculo funcionan directamente en un modelo:

```ruby
Client.count
# SELECT count(*) AS count_all FROM clients
```

O en una relación:

```ruby
Client.where(first_name: 'Ryan').count
# SELECT count(*) AS count_all FROM clients WHERE (first_name = 'Ryan')
```

También puede utilizar varios métodos de búsqueda en una relación para realizar cálculos complejos:

```ruby
Client.includes("orders").where(first_name: 'Ryan', orders: { status: 'received' }).count
```

Que ejecutará:

```ruby
SELECT count(DISTINCT clients.id) AS count_all FROM clients
  LEFT OUTER JOIN orders ON orders.client_id = clients.id WHERE
  (clients.first_name = 'Ryan' AND orders.status = 'received')
```



### 21.1 Count

Si desea ver cuántos registros hay en la tabla de su modelo, puede llamar a `Client.count` y devolverá el número. Si desea ser más específico y encontrar todos los clientes con su edad presentes en la base de datos puede utilizar `Client.count(:age)`.



### 21.2 Average

Si desea ver el promedio de un determinado número en una de sus tablas, puede llamar al método `average` de la clase que se relaciona con la tabla. Esta llamada al método se verá así:

```ruby
Client.average("orders_count")
```

Esto devolverá un número \(posiblemente un número de punto flotante tal como 3.14159265\) que representa el valor promedio en el campo.



### 21.3 Minimum

Si desea encontrar el valor mínimo de un campo en su tabla, puede llamar al método `minimum` de la clase que se relaciona con la tabla. Esta llamada al método se verá así:

```ruby
Client.minimum("age")
```



### 21.4 Maximum

Si desea encontrar el valor máximo de un campo en su tabla, puede llamar al método `maximum` de la clase que se relaciona con la tabla. Esta llamada al método se verá así:

```ruby
Client.maximum("age")
```



### 21.5 Sum

Si desea encontrar la suma de un campo para todos los registros de su tabla, puede llamar al método `sum` en la clase que se relaciona con la tabla. Esta llamada al método se verá así:

```ruby
Client.sum("orders_count")
```



