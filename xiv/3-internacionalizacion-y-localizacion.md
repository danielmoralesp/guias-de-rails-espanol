# 3- Internacionalización y localización

¡Ok! Ahora ha _**inicializado**_ la compatibilidad con I18n para su aplicación de Ruby on Rails y le ha indicado qué idioma usar y cómo preservarla entre las solicitudes.

Luego necesitamos _**internacionalizar**_ nuestra aplicación abstrayendo cada elemento específico de la configuración regional. Finalmente, necesitamos _**localizarlo**_ proporcionando las traducciones necesarias para estos resúmenes.

Dado el siguiente ejemplo:

```ruby
# config/routes.rb
Rails.application.routes.draw do
  root to: "home#index"
end
```

```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  before_action :set_locale

  def set_locale
    I18n.locale = params[:locale] || I18n.default_locale
  end
end
```

```ruby
# app/controllers/home_controller.rb
class HomeController < ApplicationController
  def index
    flash[:notice] = "Hello Flash"
  end
end
```

```ruby
# app/views/home/index.html.erb
<h1>Hello World</h1>
<p><%= flash[:notice] %></p>
```

![](/assets/Captura de pantalla 2017-07-12 a las 2.21.42 p.m..png)

### 3.1 Abstrayendo el código a localizar

Hay dos cadenas en nuestro código que están en inglés y que los usuarios renderizarán en nuestra respuesta \("Hello Flash" y "Hello World"\). Para internacionalizar este código, estas cadenas deben ser reemplazadas por llamadas al ayudante  `#t` de Rails con una clave apropiada para cada cadena:

```ruby
# app/controllers/home_controller.rb
class HomeController < ApplicationController
  def index
    flash[:notice] = t(:hello_flash)
  end
end
```

```ruby
# app/views/home/index.html.erb
<h1><%=t :hello_world %></h1>
<p><%= flash[:notice] %></p>
```

Ahora, cuando se muestre esta vista, mostrará un mensaje de error que indica que faltan las traducciones de las claves `:hello_world` y `:hello_flash`.

![](/assets/Captura de pantalla 2017-07-12 a las 2.35.23 p.m..png)

> Rails añade un método auxiliar `t`\(translate\) a sus vistas de modo que no necesite deletrear `I18n.t` todo el tiempo. Además, este ayudante capturará las traducciones perdidas y envolverá el mensaje de error resultante en un `<span class = "translation_missing">`.

### 3.2 Proporcionar traducciones para cadenas internacionalizadas

Agregue las traducciones faltantes a los archivos del diccionario de traducción:

```ruby
# config/locales/en.yml
en:
  hello_world: Hello world!
  hello_flash: Hello flash!

# config/locales/pirate.yml
pirate:
  hello_world: Ahoy World
  hello_flash: Ahoy Flash
```

Debido a que `default_locale` no ha cambiado, las traducciones usan `:en` locale y la respuesta procesa el string en inglés:

![](/assets/Captura de pantalla 2017-07-14 a las 2.14.53 p.m..png)

Si la configuración regional se establece a través de la URL de la configuración regional "pirata" \(`http://localhost:3000?locale=pirate`\), la respuesta procesa los strings "piratas":

![](/assets/Captura de pantalla 2017-07-14 a las 2.17.07 p.m..png)

> Debe reiniciar el servidor cuando agrega nuevos archivos de configuración regional.

Puede utilizar archivos YAML \(`.yml`\) o Ruby \(`.rb`\) para almacenar sus traducciones en SimpleStore. YAML es la opción preferida entre los desarrolladores de Rails. Sin embargo, tiene una gran desventaja. YAML es muy sensible a espacios en blanco y caracteres especiales, por lo que la aplicación no puede cargar su diccionario correctamente. Los archivos Ruby bloquearán su aplicación en la primera solicitud, por lo que puede encontrar fácilmente lo que está mal. \(Si encuentra algún "problema extraño" con los diccionarios YAML, intente colocar la parte relevante de su diccionario en un archivo Ruby\).

Si sus traducciones se almacenan en archivos YAML, ciertas claves deben escaparse. Son:

* true, por, yes
* false, por, no

Ejemplos:

```ruby
# config/locales/en.yml
en:
  success:
    'true':  'True!'
    'on':    'On!'
    'false': 'False!'
  failure:
    true:    'True!'
    off:     'Off!'
    false:   'False!'
```

```ruby
I18n.t 'success.true' # => 'True!'
I18n.t 'success.on' # => 'On!'
I18n.t 'success.false' # => 'False!'
I18n.t 'failure.false' # => Translation Missing
I18n.t 'failure.off' # => Translation Missing
I18n.t 'failure.true' # => Translation Missing
```

### 3.3 Pasando variables a las traducciones

Una consideración clave para internacionalizar con éxito una aplicación es evitar hacer suposiciones incorrectas acerca de las reglas gramaticales al abstraer el código localizado. Las reglas de gramática que parecen fundamentales en una localización pueden no ser válidas en otra.

La abstracción incorrecta se muestra en el siguiente ejemplo, donde se hacen suposiciones sobre el ordenamiento de las diferentes partes de la traducción. Tenga en cuenta que Rails proporciona un ayudante `number_to_currency` para manejar el siguiente caso.

```ruby
# app/views/products/show.html.erb
<%= "#{t('currency')}#{@product.price}" %>
```

```ruby
# config/locales/en.yml
en:
  currency: "$"
 
# config/locales/es.yml
es:
  currency: "€"
```

Si el precio del producto es 10 entonces la traducción correcta para el español es "10€" en lugar de "€10", pero la abstracción no puede darlo.

Para crear una abstracción adecuada, la gema I18n viene con una función llamada interpolación de variables que le permite utilizar variables en las definiciones de traducción y pasar los valores de estas variables al método de traducción.

La abstracción apropiada se muestra en el siguiente ejemplo:

```ruby
# app/views/products/show.html.erb
<%= t('product_price', price: @product.price) %>
```

```ruby
# config/locales/en.yml
en:
  product_price: "$%{price}"
 
# config/locales/es.yml
es:
  product_price: "%{price} €"
```

Todas las decisiones gramaticales y de puntuación se toman en la definición misma, por lo que la abstracción puede dar una traducción correcta.

Las palabras clave predeterminadas y de ámbito están reservadas y no se pueden utilizar como nombres de variables. Si se utiliza, se genera una excepción `I18n::ReservedInterpolationKey`. Si una traducción espera una variable de interpolación, pero esto no se ha pasado a `#translate`, se genera una excepción `I18n::MissingInterpolationArgument` .

### 3.4 Adición de formatos de fecha / hora

¡Ok! Ahora vamos a agregar una marca de tiempo a la vista, por lo que también puede mostrar la función de localización de fecha y hora. Para localizar el formato de hora, pasa el objeto `Time` a `I18n.l` o \(preferiblemente\) usa el ayudante `#l `de Rails. Puede elegir un formato pasando la opción `:format` - por defecto se usa el formato `:default`.

```ruby
# app/views/home/index.html.erb
<h1><%=t :hello_world %></h1>
<p><%= flash[:notice] %></p>
<p><%= l Time.now, format: :short %></p>
```

Y en nuestro archivo de traducciones piratas vamos a añadir un formato de hora \(ya está ahí en los valores predeterminados de Rails en el inglés\):

```ruby
# config/locales/pirate.yml
pirate:
  time:
    formats:
      short: "arrrround %H'ish"
```

Así que eso te daría:

![](/assets/Captura de pantalla 2017-07-14 a las 2.28.09 p.m..png)

> En este caso puede que tenga que agregar algunos formatos más de fecha/hora para hacer que el backend `I18n` funcione como se esperaba \(al menos para el escenario "pirata"\). Por supuesto, hay una gran posibilidad de que alguien ya haya hecho todo el trabajo al traducir los valores predeterminados de Rails para su entorno. Consulte el repositorio rails-i18n en GitHub para obtener un archivo de varios archivos de configuración regional. Cuando pones tal archivo\(s\) en el directorio `config/locales/`, automáticamente estarán listos para su uso.

### 3.5 Reglas de inflexión para otros locales

Rails le permite definir reglas de inflexión \(como reglas para la singularización y la pluralización\) para idiomas distintos del inglés. En `config/initializers/inflections.rb`, puede definir estas reglas para varias localidades. El inicializador contiene un ejemplo predeterminado para especificar reglas adicionales para el inglés; Siga ese formato para otras configuraciones regionales como mejor le parezca.

### 3.6 Vistas localizadas

Digamos que usted tiene un BooksController en su aplicación. La acción `index` procesa contenido en la plantilla `app/views/books/index.html.erb`. Cuando se coloca una variante localizada de esta plantilla: `index.es.html.erb` en el mismo directorio, Rails procesará contenido en esta plantilla, cuando la configuración regional esté establecida en `:es`. Cuando la configuración regional se establece en la configuración regional predeterminada, se utilizará la vista genérica `index.html.erb`. \(Las versiones de Future Rails pueden traer esta localización automagic a los assets en `public`, etc.\)

Puede utilizar esta función por ejemplo, cuando se trabaja con una gran cantidad de contenido estático, lo que sería torpe para poner dentro de los diccionarios YAML o Ruby. Tenga en cuenta, sin embargo, que cualquier cambio que le gustaría hacer más tarde a la plantilla debe propagarse a todos ellos.









