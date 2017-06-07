# 6.14 CsrfHelper

Devuelve las etiquetas meta "`csrf-param`" y "`csrf-token`" con el nombre del parámetro de protección de falsificación de solicitud de cross-site y símbolo, respectivamente.

```ruby
<%= csrf_meta_tags %>
```

> Los formularios regulares generan campos ocultos para que no utilicen estas etiquetas. Se pueden encontrar más detalles en la Guía de seguridad de Rails.



