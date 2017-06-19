# 13- Filtrado de logs

Rails mantiene un archivo de logs para cada entorno en la carpeta log. Estos son extremadamente útiles cuando se depura lo que realmente está pasando en su aplicación, pero en una aplicación en vivo puede que no quiera que cada bit de información sea almacenado en el archivo de logs.

### 13.1 Filtrado de parámetros

Puede filtrar los parámetros de solicitudes sensibles de sus archivos de logs agregándolos a `config.filter_parameters` en la configuración de la aplicación. Estos parámetros se marcarán \[FILTERED\] en el log.

```ruby
config.filter_parameters << :password
```

Los parámetros proporcionados se filtrarán mediante una expresión regular coincidente parcial. Rails añade el valor predeterminado `:password` en el inicializador apropiado \(`initialalizers/filter_parameter_logging.rb`\) y se preocupa por los parámetros típicos de la aplicación `password` y `password_confirmation`.

### 13.2 Redirecciona el filtrado

A veces es deseable filtrar desde archivos de logs algunas ubicaciones sensibles a las que está redirigiendo la aplicación. Puede hacerlo usando la opción de configuración `config.filter_redirect`:

```ruby
config.filter_redirect << 's3.amazonaws.com'
```

Puede establecerla en una cadena, una Regex o un array de ambas.

```ruby
config.filter_redirect.concat ['s3.amazonaws.com', /private_path/]
```

Las URL coincidentes se marcarán como "\[FILTERED\]".

