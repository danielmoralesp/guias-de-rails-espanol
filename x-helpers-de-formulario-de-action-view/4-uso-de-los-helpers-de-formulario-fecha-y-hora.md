# 4- Uso de los helpers de formulario Fecha y Hora

Puede optar por no utilizar los helpers de formulario que generan campos de entrada de fecha y hora en `HTML5` y utilizar los helpers alternativos de fecha y hora. Estos helpers de fecha y hora difieren de todos los demás helpers de formulario en dos aspectos importantes:

* Las fechas y los tiempos no son representables por un único elemento input. En su lugar, tiene varios, uno para cada componente \(year, month, day etc.\) y, por lo tanto, no hay un valor único en su `params` hash con su fecha o hora.
* Otros helpers usan el sufijo `_tag` para indicar si un helper es un helper de barebones o uno que opera con objetos de un modelo. En fechas y horas, `select_date`, `select_time` y `select_datetime` son los helpers de barebones, y `date_select`, `time_select` y `datetime_select` son los helpers de objetos de un modelo equivalente.

Ambas familias de helpers crearán una serie de cajas de seleccion para los diferentes componentes \(año, mes, día, etc.\).



### 4.1 Helpers Barebones

La familia `select_ *` de helpers toma como su primer argumento una instancia de `Date`, `Time` y `DateTime` que se utiliza como el valor seleccionado actualmente. Puede omitir este parámetro, en cuyo caso se utilizará la fecha actual. Por ejemplo:

```ruby
<%= select_date Date.today, prefix: :start_date %>
```

Salidas \(con valores de opción reales omitidos por brevedad\)

```ruby
<select id="start_date_year" name="start_date[year]"> ... </select>
<select id="start_date_month" name="start_date[month]"> ... </select>
<select id="start_date_day" name="start_date[day]"> ... </select>
```

Las entradas anteriores darían lugar a `params[:start_date]` siendo un hash con claves `:year`, `:month`, `:day`. Para obtener un objeto `Date`, `Time` o `DateTime` real, tendría que extraer estos valores y pasarlos al constructor apropiado, por ejemplo:

```ruby
Date.civil(params[:start_date][:year].to_i, params[:start_date][:month].to_i, params[:start_date][:day].to_i)
```



