# 5. Saltando Callbacks

Así como con las validaciones, es también posible saltarse callbacks por el uso de los siguientes métodos:

```ruby
decrement
decrement_counter
delete
delete_all
increment
increment_counter
toggle
touch
update_column
update_columns
update_all
update_counters
```

Estos métodos deben ser utilizados con precaución, porque importantes reglas de negocio y lógica de la aplicación pueden ser mantenidas en los callbacks. Pasar por encima de ellas sin entender las implicaciones potenciales podrían dejar inválidos los datos.

