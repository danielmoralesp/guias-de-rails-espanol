# 6.6 DateHelper



### 6.6.1 date\_select

Devuelve un conjunto de etiquetas de selección \(una por año, mes y día\) preseleccionadas para acceder a un atributo basado-en-fecha especificado.

```ruby
date_select("article", "published_on")
```



### 6.6.2 datetime\_select

Devuelve un conjunto de etiquetas de selección \(una por año, mes, día, hora y minuto\) preseleccionadas para acceder a un atributo de fecha y hora especificado.

```ruby
datetime_select("article", "published_on")
```



### 6.6.3 distance\_of\_time\_in\_words

Informa la distancia aproximada en el tiempo entre dos objetos `Time` o `Date` o números enteros como segundos. Establezca `include_seconds` a `true` si desea obtener aproximaciones más detalladas.

```ruby
distance_of_time_in_words(Time.now, Time.now + 15.seconds)        # => less than a minute
distance_of_time_in_words(Time.now, Time.now + 15.seconds, include_seconds: true)  # => less than 20 seconds
```



### 6.6.4 select\_date

Devuelve un conjunto de etiquetas de selección HTML \(uno para año, mes y día\) preseleccionado con la fecha proporcionada.

```ruby
# Genera una fecha seleccionada que se ajusta por defecto a la fecha proporcionada (seis días después de hoy)
select_date(Time.today + 6.days)

# Genera una fecha seleccionada por defecto a la fecha actual (sin fecha especificada)
select_date()
```



### 6.6.5 select\_datetime

Devuelve un conjunto de etiquetas de selección de `HTML` \(una para el año, el mes, el día, la hora y el minuto\) preseleccionadas con la fecha y hora previstas.

```ruby
# Genera una fecha y hora que selecciona por defecto la fecha y hora que se proporciona (cuatro días después de hoy)
select_datetime(Time.now + 4.days)

# Genera una fecha y hora que selecciona por defecto a la fecha de hoy (no hay fecha especificada)
select_datetime()
```



### 6.6.6 select\_day

Devuelve una etiqueta de selección con opciones para cada uno de los días del 1 al 31 con el día actual seleccionado.

```ruby
# Genera un campo de selección para los días cuyo valor predeterminado es el día de la fecha proporcionada
select_day(Time.today + 2.days)

# Genera un campo de selección para los días que predeterminan el número dado
select_day(5)
```

### 6.6.7 select\_hour

Devuelve una etiqueta de selección con opciones para cada una de las horas de 0 a 23 con la hora actual seleccionada.

```ruby
# Genera un campo de selección para las horas que predeterminan las horas para el tiempo proporcionado
select_hour(Time.now + 6.hours)
```





### 6.6.8 select\_minute

Devuelve una etiqueta de selección con opciones para cada uno de los minutos 0 a 59 con el minuto actual seleccionado.

```ruby
# Genera un campo de selección para los minutos que predeterminan los minutos para el tiempo proporcionado.
select_minute(Time.now + 10.minutes)
```



### 6.6.9 select\_month

Devuelve una etiqueta de selección con opciones para cada uno de los meses de enero a diciembre con el mes actual seleccionado.

```ruby
# Genera un campo de selección para meses que predetermina el mes actual
select_month(Date.today)
```



### 6.6.10 select\_second

Devuelve una etiqueta de selección con opciones para cada uno de los segundos 0 a 59 con el segundo actual seleccionado.

```ruby
# Genera un campo de selección para segundos que esta predeterminado a los segundos para el tiempo proporcionado
select_second(Time.now + 16.seconds)
```



### 6.6.11 select\_time

Devuelve un conjunto de etiquetas de selección HTML \(una por hora y minuto\).

```ruby
# Genera una selección de tiempo que se ajusta por defecto a la hora proporcionada
select_time(Time.now)
```



### 6.6.12 select\_year

Devuelve una etiqueta de selección con opciones para cada uno de los cinco años de cada lado del actual que está seleccionado. El rango de cinco años se puede cambiar usando las teclas `:start_year` y `:end_year` en las opciones.

```ruby
# Genera un campo de selección durante cinco años a cada lado de Date.today predeterminado al año actual
select_year(Date.today)
 
# Genera un campo de selección de 1900 a 2009 y el predeterminado es el año actual
select_year(Date.today, start_year: 1900, end_year: 2009)
```



### 6.6.13 time\_ago\_in\_words

Como `distance_of_time_in_words`, pero donde `to_time` está fijado a `Time.now`.

```ruby
time_ago_in_words(3.minutes.from_now)  # => 3 minutes
```



### 6.6.14 time\_select

Devuelve un conjunto de etiquetas de selección \(uno para hora, minuto y opcionalmente segundos\) preseleccionado para acceder a un atributo de tiempo específico. Los `select` se preparan para la asignación de múltiples parámetros a un objeto de Active Record.

```ruby
# Creates a time select tag that, when POSTed, will be stored in the order variable in the submitted attribute
time_select("order", "submitted")
```



