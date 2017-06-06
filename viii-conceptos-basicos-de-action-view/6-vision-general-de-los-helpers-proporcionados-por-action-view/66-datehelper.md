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















