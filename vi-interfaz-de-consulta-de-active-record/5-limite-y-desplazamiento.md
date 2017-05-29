# 5. Límite y desplazamiento

Para aplicar `LIMIT` al `SQL` disparado por el `Model.find`, puede especificar el límite utilizando los métodos `limit` y `offset` en la relación.

Puede utilizar `limit` para especificar el número de registros que se recuperarán y utilizar `offeset` para especificar el número de registros a omitir antes de comenzar a devolver los registros. Por ejemplo:

```ruby
Client.limit(5)
```

Devolverá un máximo de 5 clientes y, como no especifica ningún desplazamiento, devolverá los primeros 5 de la tabla. El `SQL` que ejecuta se ve así:

```ruby
SELECT * FROM clients LIMIT 5
```

Añadiendo `offset` a este ejemplo:

```ruby
Client.limit(5).offset(30)
```

Retornará un máximo de 5 clientes comenzando con el 31. El `SQL` se ve así:

```ruby
SELECT * FROM clients LIMIT 5 OFFSET 30
```



