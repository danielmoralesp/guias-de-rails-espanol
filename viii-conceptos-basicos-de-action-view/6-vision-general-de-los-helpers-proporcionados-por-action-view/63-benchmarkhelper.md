# 6.3 BenchmarkHelper



### 6.3.1 benchmark

Permite medir el tiempo de ejecución de un bloque en una plantilla y graba el resultado en el registro. Envuelva este bloque alrededor de operaciones costosas o posibles cuellos de botella para obtener una lectura de tiempo para la operación.

```ruby
<% benchmark "Process data files" do %>
  <%= expensive_files_operation %>
<% end %>
```

Esto añadiría algo como "Process data files \(0.34523\)" al registro, que puede utilizar para comparar los tiempos al optimizar su código.

