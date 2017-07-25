# 5- Personalización del Canalizador

### 5.1 Compresión de CSS

Una de las opciones para comprimir CSS es `YUI`. El compresor CSS de `YUI` proporciona la minificación.

La línea siguiente permite la compresión en `YUI`, y requiere la gema `yui-compresor`.

```ruby
config.assets.css_compressor = :yui
```

La otra opción para comprimir CSS si tiene instalada la gema `sass-rails`

```ruby
config.assets.css_compressor = :sass
```

### 5.2 Compresión de JavaScript

Las opciones posibles para la compresión de JavaScript son :`closure`, `:uglifier` y `:yui`. Éstos requieren el uso de las gemas `closure-compiler`, `uglifier` o `yui-compressor`  respectivamente.

El Gemfile por defecto incluye `uglifier`. Esta gema envuelve UglifyJS \(escrito para NodeJS\) en Ruby. Se comprime el código mediante la eliminación de espacios en blanco y comentarios, acortando los nombres de las variables locales, y la realización de otras micro-optimizaciones, como el cambio de las declaraciones `if` y `else`  a los operadores ternarios siempre que sea posible.

La siguiente línea invoca `uglifier` para la compresión de JavaScript.

```ruby
config.assets.js_compressor = :uglifier
```

> Necesitará un tiempo de ejecución soportado por ExecJS para poder usar `uglifier`. Si está utilizando macOS o Windows, tiene un tiempo de ejecución de JavaScript instalado en su sistema operativo.

### 5.3 Servir la versión GZipped de los recursos

De forma predeterminada, se generará la versión comprimida de los recursos compilados, junto con la versión no comprimida de los recursos. Los recursos con `Gzip` ayudan a reducir la transmisión de datos a través del cable. Puede configurar esto configurando el indicador `gzip`.

```ruby
config.assets.gzip = false # disable gzipped assets generation
```

### 5.4 Uso de su propio compresor

Los ajustes de configuración del compresor para CSS y JavaScript también pueden tomar cualquier objeto. Este objeto debe tener un método de compresión que toma una cadena como único argumento y debe devolver una cadena.

```ruby
class Transformer
  def compress(string)
    do_something_returning_a_string(string)
  end
end
```

Para habilitar esto, pase un nuevo objeto a la opción `config` en `application.rb`:

```ruby
config.assets.css_compressor = Transformer.new
```

### 5.5 Modificación del assets path

El path publico que Sprockets utiliza por defecto es `/assets`.

Esto se puede cambiar a otra cosa:

```ruby
config.assets.prefix = "/some_other_path"
```

Esta es una opción práctica si está actualizando un proyecto anterior que no se utilizó a la canalización de recursos y ya utiliza esta ruta o desea utilizar esta ruta de acceso para un recurso nuevo.

### 5.6 Encabezados X-Sendfile

El encabezado X-Sendfile es una directiva para que  el servidor web ignore la respuesta de la aplicación y, en su lugar, sirva un archivo especifico desde el disco. Esta opción está desactivada de forma predeterminada, pero se puede habilitar si su servidor lo admite. Cuando se habilita, esto le da la responsabilidad de servir el archivo al servidor web, que es más rápido. Eche un vistazo a [`send_file`](http://api.rubyonrails.org/v5.1.2/classes/ActionController/DataStreaming.html#method-i-send_file) sobre cómo usar esta función.

Apache y NGINX admiten esta opción, que se puede habilitar en `config/environments/production.rb`:

```ruby
# config.action_dispatch.x_sendfile_header = "X-Sendfile" # for Apache
# config.action_dispatch.x_sendfile_header = 'X-Accel-Redirect' # for NGINX
```

> Si se está actualizando a una aplicación existente y tiene la intención de usar esta opción, tenga cuidado de pegar esta opción de configuración sólo en `production.rb` y en cualquier otro entorno que defina con comportamiento de producción \(no `application.rb`\).

> Para más detalles, eche un vistazo a los documentos de su servidor de producción:  -[Apache](https://tn123.org/mod_xsendfile/)-[NGINX](http://wiki.nginx.org/XSendfile)





