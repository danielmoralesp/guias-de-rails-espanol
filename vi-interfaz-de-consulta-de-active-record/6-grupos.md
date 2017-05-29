# 6. Grupos

Para aplicar una cláusula `GROUP BY` al `SQL` disparado por el `finder`, puede utilizar el método de `group`.

Por ejemplo, si desea encontrar una colección de las fechas en las que se crearon los pedidos:

```ruby
Order.select("date(created_at) as ordered_date, sum(price) as total_price").group("date(created_at)")
```

Y esto le dará un único objeto `Order` para cada fecha en la que haya pedidos en la base de datos.

El `SQL` que se ejecutaría sería algo como esto:

```ruby
SELECT date(created_at) as ordered_date, sum(price) as total_price
FROM orders
GROUP BY date(created_at)
```



### 6.1 Total de elementos agrupados

Para obtener el total de elementos agrupados en una sola consulta, llame a `count` después de `group`.

```ruby
Order.group(:status).count
# => { 'awaiting_approval' => 7, 'paid' => 12 }
```

El `SQL` que se ejecutaría sería algo como esto:

```ruby
SELECT COUNT (*) AS count_all, status AS status
FROM "orders"
GROUP BY status
```



