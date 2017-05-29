# 10- Objetos de sólo lectura

Active Record proporciona el método `readonly` en una relación para rechazar explícitamente la modificación de cualquiera de los objetos devueltos. Cualquier intento de alterar un registro de sólo lectura no tendrá éxito, generando una excepción `ActiveRecord::ReadOnlyRecord`.

```ruby
client = Client.readonly.first
client.visits += 1
client.save
```

`Client` se establece explícitamente para ser un objeto de sólo lectura, el código anterior generará una excepción `ActiveRecord::ReadOnlyRecord` al llamar a` client.save` con un valor actualizado de visitas.

