# 13. Asociaciones de carga impaciente

La carga ansiosa es el mecanismo para cargar los registros asociados de los objetos devueltos por `Model.find` utilizando el menor número posible de consultas.

N + 1 problema de consultas

Considere el siguiente código, que encuentra 10 clientes e imprime sus códigos postales:

```ruby
clients = Client.limit(10)
 
clients.each do |client|
  puts client.address.postcode
end
```

Este código se ve bien a primera vista. Pero el problema está dentro del número total de consultas ejecutadas. El código anterior ejecuta 1 consulta \(para encontrar 10 clientes\) + 10 consultas adicionales \(uno por cada cliente para cargar la dirección\) = 11 consultas en total.

**Solución al problema de consultas N + 1**



