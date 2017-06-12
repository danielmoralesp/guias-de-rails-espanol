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

La opción `:prefix` es la clave utilizada para recuperar el `hash` de los componentes de fecha del `hash` de `params`. Aquí se estableció en `start_date`, si se omite se predeterminará hasta la fecha.



### 4.2 Helpers de objetos de un modelo

`select_date` no funciona bien con los formularios que se actualizan o crean objetos Active Record ya que Active Record espera que cada elemento del hash `params` corresponda a un atributo. Los helpers de objetos de un modelo para fechas y horas envían parámetros con nombres especiales; Cuando Active Record ve parámetros con tales nombres sabe que deben ser combinados con los otros parámetros y dados a un constructor apropiado al tipo de columna. Por ejemplo:

```ruby
<%= date_select :person, :birth_date %>
```

Salidas \(con valores de opción reales omitidos por brevedad\)

```ruby
<select id="person_birth_date_1i" name="person[birth_date(1i)]"> ... </select>
<select id="person_birth_date_2i" name="person[birth_date(2i)]"> ... </select>
<select id="person_birth_date_3i" name="person[birth_date(3i)]"> ... </select>
```

Que da como resultado un hash `params` como:

```ruby
{'person' => {'birth_date(1i)' => '2008', 'birth_date(2i)' => '11', 'birth_date(3i)' => '22'}}
```

Cuando se pasa a `Person.new` \(o `update`\), Active Record indica que todos estos parámetros deben usarse para construir el atributo `birth_date` y utiliza la información del sufijo para determinar en qué orden debe pasar estos parámetros a funciones como `Date.civil` .



### 4.3 Opciones comunes

Ambas familias de helpers utilizan el mismo conjunto básico de funciones para generar las etiquetas individuales de selección y por lo tanto aceptan en gran medida las mismas opciones. En particular, por defecto Rails generará opciones de año, 5 años a cada lado del año en curso. Si no es un rango apropiado, las opciones `:start_year` y `:end_year` reemplazan esto. Para obtener una lista exhaustiva de las opciones disponibles, consulte la documentación de la API.

Como regla general, debe utilizar `date_select` cuando trabaje con objetos de un modelo y `select_date` en otros casos, como un formulario de búsqueda que filtre los resultados por fecha.

> En muchos casos los date pickers incorporados son torpes ya que no ayudan al usuario en la elaboración de la relación entre la fecha y el día de la semana.



### 4.4 Componentes individuales

De vez en cuando usted necesita exhibir apenas un solo componente de la fecha tal como un año o un mes. Rails proporciona una serie de ayudantes para esto, uno para cada componente `select_year`, `select_month`, `select_day`, `select_hour`, `select_minute`, `select_second`. Estos helpers son bastante sencillos. De forma predeterminada, generarán un `input` con el nombre del componente tiempo \(por ejemplo, "year" para `select_year`, "month" para `select_month`, etc.\), aunque esto puede anularse con la opción `:field_name`. La opción `:prefix` funciona de la misma manera que hace para `select_date` y `select_time` y tiene el mismo valor predeterminado.

El primer parámetro especifica qué valor debe seleccionarse y puede ser una instancia de `Date`, `Time` o `DateTime`, en cuyo caso se extraerá el componente relevante o un valor numérico. Por ejemplo:

```ruby
<%= select_year(2009) %>
<%= select_year(Time.now) %>
```

Producirá la misma salida si el año actual es 2009 y el valor elegido por el usuario puede ser recuperado por `params[:date] [:year]`.

