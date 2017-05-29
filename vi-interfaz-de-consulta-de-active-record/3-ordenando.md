# 3. Ordenando

Para recuperar registros de la base de datos en un orden específico, puede utilizar el método `order`.

Por ejemplo, si obtiene un conjunto de registros y desea ordenarlos en orden ascendente por el campo `created_at` en su tabla:

```ruby
Client.order(:created_at)
# OR
Client.order("created_at")
```

También puede especificar `ASC` o `DESC`:

```ruby
Client.order(created_at: :desc)
# OR
Client.order(created_at: :asc)
# OR
Client.order("created_at DESC")
# OR
Client.order("created_at ASC")
```

O ordenar por múltiples campos:

```ruby
Client.order(orders_count: :asc, created_at: :desc)
# OR
Client.order(:orders_count, created_at: :desc)
# OR
Client.order("orders_count ASC, created_at DESC")
# OR
Client.order("orders_count ASC", "created_at DESC")
```

Si desea llamar `order` varias veces, los métodos `order` siguientes se añadirán al primero:

```ruby
Client.order("orders_count ASC").order("created_at DESC")
# SELECT * FROM clients ORDER BY orders_count ASC, created_at DESC
```

> Si está utilizando MySQL 5.7.5 o superior, a continuación, en la selección de campos de un conjunto de resultados utilizando métodos como `select`, `pluck` y `ids`; El método `order` generará una excepción `ActiveRecord::StatementInvalid` a menos que el campo\(s\) utilizado\(s\) en la cláusula `order` estén incluidos en la lista de selección. Consulte la siguiente sección para seleccionar campos del conjunto de resultados.





