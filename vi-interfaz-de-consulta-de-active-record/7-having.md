# 7. Having

`SQL` utiliza la cláusula `HAVING` para especificar las condiciones en los campos `GROUP BY`. Puede agregar la cláusula `HAVING` al `SQL` disparado por el `Model.find` añadiendo el método `having` al `find`.

Por ejemplo:

```ruby
Order.select("date(created_at) as ordered_date, sum(price) as total_price").
  group("date(created_at)").having("sum(price) > ?", 100)
```

El `SQL` que se ejecutaría sería algo como esto:

```ruby
SELECT date(created_at) as ordered_date, sum(price) as total_price
FROM orders
GROUP BY date(created_at)
HAVING sum(price) > 100
```

Esto devuelve la fecha y el precio total de cada objeto de pedido, agrupados por el día en que fueron pedidos y donde el precio es superior a $ 100.



