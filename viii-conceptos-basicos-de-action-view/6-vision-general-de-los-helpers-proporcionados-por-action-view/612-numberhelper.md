# 6.12 NumberHelper

Proporciona métodos para convertir números en cadenas formateadas. Los métodos se proporcionan para números de teléfono, moneda, porcentaje, precisión, notación posicional y tamaño del archivo.



### 6.12.1 number\_to\_currency

Formatea un número en una cadena de moneda \(por ejemplo, $ 13.65\).

```ruby
number_to_currency(1234567890.50) # => $1,234,567,890.50
```



### 6.12.2 number\_to\_human\_size

Formatea los bytes de tamaño en una representación más comprensible; Útil para reportar el tamaño de los archivos a los usuarios.

```ruby
number_to_human_size(1234)          # => 1.2 KB
number_to_human_size(1234567)       # => 1.2 MB
```



### 6.12.3 number\_to\_percentage

Formatea un número como una cadena de porcentaje.

```ruby
number_to_percentage(100, precision: 0)        # => 100%
```



### 6.12.4 number\_to\_phone

Formatea un número en un número de teléfono \(EE.UU. por defecto\).

```ruby
number_to_phone(1235551234) # => 123-555-1234
```



### 6.12.5 number\_with\_delimiter

Formatea un número con miles agrupados usando un delimitador.

```ruby
number_with_delimiter(12345678) # => 12,345,678
```



### 6.12.6 number\_with\_precision

Formatea un número con el nivel de precisión especificado, que por defecto es 3.

```ruby
number_with_precision(111.2345)     # => 111.235
number_with_precision(111.2345, precision: 2)  # => 111.23
```



