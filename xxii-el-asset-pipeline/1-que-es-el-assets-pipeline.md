# 1- ¿Qué es el la Canalización de Recursos?

La canalización de recursos proporciona un framework para concatenar y minificar o comprimir los recursos de JavaScript y CSS. También añade la capacidad de escribir estos recursos en otros lenguajes y preprocesadores como CoffeeScript, Sass y ERB. Permite que los recursos de su aplicación se combinen automáticamente con recursos de otras gemas. Por ejemplo, `jquery-rails` incluye una copia de `jquery.js` y habilita las funciones de AJAX en Rails.

La canalización de recursos está implementada por la gema `sprockets-rails` y está habilitada de forma predeterminada. Puede desactivarlo mientras crea una nueva aplicación pasando la opción `--skip-sprockets`.

```ruby
rails new appname --skip-sprockets
```

Rails añade automáticamente las gemas `sass-rails`, `coffee-rails` y `uglifier` a su Gemfile, que son utilizados por Sprockets para comprimir los recursos:

```ruby
gem 'sass-rails'
gem 'uglifier'
gem 'coffee-rails'
```

El uso de la opción `--skip-sprockets` evitará que Rails los agregue a su Gemfile, por lo que si posteriormente desea habilitar la canalización de los recursos, tendrá que agregar esas gemas a su Gemfile. Además, la creación de una aplicación con la opción `--skip-sprockets` generará un archivo `config/application.rb` ligeramente diferente, con una instrucción `require` para el railtie de los sprockets que estará comentado. Tendrá que quitar el operador de comentario en esa línea para habilitar más tarde la canalización de los recursos:

```ruby
# require "sprockets/railtie"
```

Para establecer métodos de compresión de recursos, establezca las opciones de configuración apropiadas en `production.rb - config.assets.css_compressor` para su CSS y `config.assets.js_compressor` para su JavaScript:

```ruby
config.assets.css_compressor = :yui
config.assets.js_compressor = :uglifier
```

> La gema `sass-rails` se utiliza automáticamente para la compresión del CSS si se incluye en el archivo Gemfile y no se establece ninguna opción `config.assets.css_compressor`.

### 1.1 Características principales

La primera característica del pipeline es concatenar los recursos, lo que puede reducir el número de solicitudes que hace un navegador para renderizar una página web. Los navegadores web están limitados en el número de solicitudes que pueden realizar en paralelo, por lo que un menor número de solicitudes puede significar una carga más rápida para su aplicación.

Sprockets concatena todos los archivos JavaScript en un archivo master `.js` y todos los archivos CSS en un archivo master `.css`. Como veremos más adelante en esta guía, puede personalizar esta estrategia para agrupar archivos de la forma que desee. En producción, Rails inserta una huella digital SHA256 en cada nombre de archivo para que el archivo sea almacenado en caché por el navegador web. Puede invalidar la memoria caché alterando esta huella digital, que ocurre automáticamente cada vez que cambia el contenido del archivo.

La segunda característica de la canalización de archivos es la minificación o compresión de los recursos. Para los archivos CSS, esto se hace mediante la eliminación de espacios en blanco y comentarios. Para JavaScript, se pueden aplicar procesos más complejos. Puede elegir entre un conjunto de opciones integradas o especificar las propias.

La tercera característica de la canalización de archivos es que permite codificar los recursos a través de un lenguaje de nivel superior, con la precompilación hasta los recursos reales. Los lenguajes admitidos incluyen Sass para CSS, CoffeeScript para JavaScript y ERB para ambos de forma predeterminada.

### 1.2 ¿Qué es la huella dactilar y por qué debería importarme?

La huella dactilar \(fingerprint\) es una técnica que hace que el nombre de un archivo dependa del contenido del archivo. Cuando cambia el contenido del archivo, también se cambia el nombre del archivo. Para contenido que es estático o cambiado con poca frecuencia, esto proporciona una manera fácil de saber si dos versiones de un archivo son idénticas, incluso entre diferentes servidores o fechas de implementación.

Cuando un nombre de archivo es único y basado en su contenido, los encabezados HTTP se pueden configurar para fomentar cachés en todas partes \(ya sea en el CDN, en el ISP, en equipos de red o en navegadores web\) para mantener su propia copia del contenido. Cuando se actualiza el contenido, la huella digital cambiará. Esto hará que los clientes remotos soliciten una nueva copia del contenido. Esto se conoce generalmente como anulación de antememoria.

La técnica Sprockets usada para la huella dactilar es insertar un hash del contenido en el nombre, generalmente al final. Por ejemplo, un archivo CSS `global.css`

```ruby
global-908e25f4bf641868d8683022a5b62f54.css
```

Esta es la estrategia adoptada por la canalización de recursos de Rails.

La vieja estrategia de Rails era añadir un query string basado en la fecha de cada recurso vinculado con un helper incorporado. La fuente del código generado se veía así:

```ruby
/stylesheets/global.css?1309495796
```

La estrategia query string tiene varias desventajas:

1. **No todas las cachés almacenarán de forma fiable el contenido donde el nombre de archivo sólo difiere según los parámetros de consulta**
   [Steve Souders recomienda](http://www.stevesouders.com/blog/2008/08/23/revving-filenames-dont-use-querystring/), "... evitando una cadena de consulta para recursos en caché". Encontró que en este caso el 5-20% de las peticiones no serán almacenadas en caché. Las cadenas de consulta en particular no funcionan en absoluto con algunas CDN para la invalidación de caché.

2. **El nombre de archivo puede cambiar entre nodos en entornos de varios servidores.**  
   La cadena de consulta predeterminada en Rails 2.x se basa en el tiempo de modificación de los archivos. Cuando los recursos se despliegan en un clúster, no hay garantía de que las marcas de tiempo sean las mismas, lo que dará lugar a que se utilicen valores diferentes dependiendo del servidor que administre la solicitud.

3. **Demasiada invalidación de caché**  
   Cuando los recursos estáticos se implementan con cada nueva versión de código, cambia el `mtime` \(tiempo de última modificación\) de todos estos archivos, forzando a todos los clientes remotos a buscarlos de nuevo, incluso cuando el contenido de esos activos no ha cambiado.

La huella dactilar soluciona estos problemas evitando los query string y asegurando que los nombres de archivo sean consistentes en función de su contenido.

La huella dactilar está habilitada de forma predeterminada para los entornos de desarrollo y producción. Puede habilitarlo o deshabilitarlo en su configuración mediante la opción `config.assets.digest`.

Más lectura:

* [Optimize catching](https://developers.google.com/speed/docs/insights/LeverageBrowserCaching?csw=1)
* [Revving Filenames: don't use querystring](http://www.stevesouders.com/blog/2008/08/23/revving-filenames-dont-use-querystring/)













