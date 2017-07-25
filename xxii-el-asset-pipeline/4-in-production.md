# 4- In Production

En el entorno de producción, Sprockets utiliza el esquema de huellas dactilares descrito anteriormente. Por defecto, Rails asume que los recursos se han precompilado y serán servidos como recursos estáticos por su servidor web.

Durante la fase de precompilación se genera un SHA256 a partir del contenido de los archivos compilados, e insertado en los nombres de los ficheros a medida que se escriben en el disco. Estos nombres de fingerprinted son utilizados por los ayudantes de Rails en lugar del nombre del manifiesto.

Por ejemplo:

```html
<%= javascript_include_tag "application" %>
<%= stylesheet_link_tag "application" %>
```

Genera algo como esto:

```html
<script src="/assets/application-908e25f4bf641868d8683022a5b62f54.js"></script>
<link href="/assets/application-4dd5b109ee3439da54f5bdfd78a80473.css" media="screen" rel="stylesheet" />
```

> Con el canalizador de recursos las opciones `:cache` y `:concat` ya no se utilizan, borre estas opciones de `javascript_include_tag` y `stylesheet_link_tag`.

El comportamiento de las huellas dactilares es controlado por la opción de inicialización `config.assets.digest` \(que por defecto es `true`\).

> En circunstancias normales, la opción predeterminada `config.assets.digest` no debe cambiarse. Si no hay `digest` en los nombres de archivo y se establecen encabezados de futuro lejano, los clientes remotos nunca harán `refetch` a los archivos cuando su contenido cambia.

### 4.1 Precompilación de recursos

Rails viene incluido con una tarea para compilar los manifiestos de recursos y otros archivos en la canalización.

Los recursos compilados se escriben en la ubicación especificada en `config.assets.prefix`. De forma predeterminada, este es el directorio `/assets`.

Puede llamar a esta tarea en el servidor durante la implementación para crear versiones compiladas de sus recursos directamente en el servidor. Consulte la siguiente sección para obtener información sobre cómo compilar localmente.

La tarea es:

```ruby
$ RAILS_ENV=production bin/rails assets:precompile
```

Capistrano \(v2.15.1 y superior\) incluye una receta para manejar esto en el despliegue. Agregue la siguiente línea a `Capfile`:

```ruby
load 'deploy/assets'
```

Esto enlaza la carpeta especificada en `config.assets.prefix` a `shared/assets`. Si ya utiliza esta carpeta compartida, tendrá que escribir su propia tarea de despliegue.

Es importante que esta carpeta se comparta entre implementaciones para que las páginas remitidas remotamente que hacen referencia a los recursos compilados antiguos funcionen durante la vida de la página almacenada en caché.

El complemento predeterminado para la compilación de archivos incluye `application.js`, `application.css` y todos los archivos que no sean JS/CSS \(esto incluirá todos los recursos de imagen automáticamente\) desde las carpetas `app/assets`, incluidas las gemas:

```ruby
[ Proc.new { |filename, path| path =~ /app\/assets/ && !%w(.js .css).include?(File.extname(filename)) },
/application.(css|js)$/ ]
```

> El matcher \(y otros miembros del array de precompilación, véase más adelante\) se aplica a los nombres de archivos compilados finales. Esto significa que cualquier cosa que se compila a JS/CSS se excluye, así como archivos crudos JS/CSS; Por ejemplo, los archivos `.coffee` y `.scss` no se incluyen automáticamente al compilarlos en JS/CSS.

Si tiene otros manifiestos u hojas de estilo individuales y archivos de JavaScript para incluir, puede agregarlos a la matriz de precompilación en `config/initializers/assets.rb`:

```ruby
Rails.application.config.assets.precompile += %w( admin.js admin.css )
```

> Siempre especifique un nombre de archivo compilado esperado que termine con `.js` o `.css`, incluso si desea agregar archivos Sass o CoffeeScript al array de precompilación.

La tarea también genera un archivo `.sprockets-manifest-md5hash.json` \(donde `md5hash` es un hash `MD5`\) que contiene una lista con todos sus recursos y sus respectivas huellas dactilares. Esto es utilizado por los métodos auxiliares de Rails para evitar entregar las solicitudes de mapeo a Sprockets. Un archivo de manifiesto típico se ve así:

```ruby
{"files":{"application-aee4be71f1288037ae78b997df388332edfd246471b533dcedaa8f9fe156442b.js":{"logical_path":"application.js","mtime":"2016-12-23T20:12:03-05:00","size":412383,
"digest":"aee4be71f1288037ae78b997df388332edfd246471b533dcedaa8f9fe156442b","integrity":"sha256-ruS+cfEogDeueLmX3ziDMu39JGRxtTPc7aqPn+FWRCs="},
"application-86a292b5070793c37e2c0e5f39f73bb387644eaeada7f96e6fc040a028b16c18.css":{"logical_path":"application.css","mtime":"2016-12-23T19:12:20-05:00","size":2994,
"digest":"86a292b5070793c37e2c0e5f39f73bb387644eaeada7f96e6fc040a028b16c18","integrity":"sha256-hqKStQcHk8N+LA5fOfc7s4dkTq6tp/lub8BAoCixbBg="},
"favicon-8d2387b8d4d32cecd93fa3900df0e9ff89d01aacd84f50e780c17c9f6b3d0eda.ico":{"logical_path":"favicon.ico","mtime":"2016-12-23T20:11:00-05:00","size":8629,
"digest":"8d2387b8d4d32cecd93fa3900df0e9ff89d01aacd84f50e780c17c9f6b3d0eda","integrity":"sha256-jSOHuNTTLOzZP6OQDfDp/4nQGqzYT1DngMF8n2s9Dto="},
"my_image-f4028156fd7eca03584d5f2fc0470df1e0dbc7369eaae638b2ff033f988ec493.png":{"logical_path":"my_image.png","mtime":"2016-12-23T20:10:54-05:00","size":23414,
"digest":"f4028156fd7eca03584d5f2fc0470df1e0dbc7369eaae638b2ff033f988ec493","integrity":"sha256-9AKBVv1+ygNYTV8vwEcN8eDbxzaequY4sv8DP5iOxJM="}},
"assets":{"application.js":"application-aee4be71f1288037ae78b997df388332edfd246471b533dcedaa8f9fe156442b.js",
"application.css":"application-86a292b5070793c37e2c0e5f39f73bb387644eaeada7f96e6fc040a028b16c18.css",
"favicon.ico":"favicon-8d2387b8d4d32cecd93fa3900df0e9ff89d01aacd84f50e780c17c9f6b3d0eda.ico",
"my_image.png":"my_image-f4028156fd7eca03584d5f2fc0470df1e0dbc7369eaae638b2ff033f988ec493.png"}}
```

La ubicación predeterminada para el manifiesto es la raíz de la ubicación especificada en `config.assets.prefix` \('`/assets`' por defecto\).

Si faltan archivos precompilados en producción, obtendrás una excepción `Sprockets::Helpers::RailsHelper::AssetPaths::AssetNotPrecompiledError` que indica el nombre de los archivos que faltan.

#### 4.1.1 Far-future caduca el encabezado

Existen recursos precompilados en el sistema de archivos y son servidos directamente por su servidor web. No tienen cabeceras de futuro por defecto, por lo que para obtener el beneficio de las huellas dactilares tendrá que actualizar la configuración del servidor para agregar esos encabezados.

Para Apache:

```ruby
# The Expires* directives requires the Apache module
# `mod_expires` to be enabled.
<Location /assets/>
  # Use of ETag is discouraged when Last-Modified is present
  Header unset ETag
  FileETag None
  # RFC says only cache for 1 year
  ExpiresActive On
  ExpiresDefault "access plus 1 year"
</Location>
```

Para NGINX:

```ruby
location ~ ^/assets/ {
  expires 1y;
  add_header Cache-Control public;
 
  add_header ETag "";
}
```

### 4.2 Precompilación local

Hay varias razones por las que puede que desee precompilar sus recursos localmente. Entre ellos están:

* Es posible que no tenga acceso de escritura a su sistema de archivos en producción.
* Es posible que esté desplegando en más de un servidor y que quiera evitar la duplicación de trabajo.
* Es posible que esté realizando despliegues frecuentes que no incluyen cambios de recursos.

La compilación local le permite asignar los archivos compilados al control de código fuente e implementarlos de forma normal.

Hay tres advertencias:

* No debe ejecutar la tarea de despliegue de Capistrano que precompila los recursos.
* Debe asegurarse de que los compresores o minifiers necesarios estén disponibles en su sistema de desarrollo.
* Debe cambiar la siguiente configuración de la aplicación:

En `config/environments/development.rb`, coloque la siguiente línea:

```ruby
config.assets.prefix = "/dev-assets"
```

El cambio de `prefix` hace que los Sprockets usen una URL diferente para servir recurso en modo de desarrollo y pasen todas las solicitudes a los Sprockets. El prefijo todavía está establecido en `/assets` en el entorno de producción. Sin este cambio, la aplicación serviría los recursos precompilados de `/assets` en desarrollo y no vería ningún cambio local hasta que compile los recursos de nuevo.

En la práctica, esto le permitirá precompilar localmente, tener esos archivos en su árbol de trabajo y confiar esos archivos al control de código fuente cuando sea necesario. El modo de desarrollo funcionará como se esperaba.

### 4.3 Compilación en vivo

En algunas circunstancias puede que desee utilizar la compilación en vivo. En este modo todas las solicitudes de recursos en el canalizador son manejadas directamente por los Sprockets.

Para activar esta opción:

```ruby
config.assets.compile = true
```

En la primera solicitud los recursos se compilan y almacenan en caché como se describe en el entorno de desarrollo anteriormente mencionado y los nombres de manifiesto utilizados en los helpers se modifican para incluir el hash SHA256.

Sprockets también establece el encabezado `HTTP Cache-Control` a `max-age=31536000`. Esto indica que todas las cachés entre su servidor y el navegador del cliente que este contenido \(el archivo que se sirve\) se puede almacenar en caché durante 1 año. El efecto de esto es reducir el número de solicitudes para este recurso de su servidor; El recurso tiene una buena probabilidad de estar en la caché del navegador local o en alguna caché intermedia.

Este modo utiliza más memoria, tiene menor rendimiento que el predeterminado y no se recomienda.

Si va a implementar una aplicación de producción en un sistema sin ningún tiempo de ejecución de JavaScript preexistente, puede agregar uno a su archivo Gemfile:

```ruby
group :production do
  gem 'therubyracer'
end
```

### 4.4 CDNs

CDN es la sigla de Content Delivery Network, diseñada principalmente para almacenar en caché recursos en todo el mundo, de forma que cuando un navegador solicita el recurso, una copia en caché estará geográficamente cerca de ese navegador. Si está sirviendo recursos directamente desde su servidor Rails en producción, la mejor práctica es usar un CDN delante de su aplicación.

Un patrón común para usar un CDN es configurar su aplicación de producción como el servidor "de origen". Esto significa que cuando un navegador solicita un recurso de la CDN y hay una falta de caché, se tomará el archivo de su servidor sobre la marcha y luego en la caché. Por ejemplo, si está ejecutando una aplicación de Rails en `example.com `y tiene un CDN configurado en `mycdnsubdomain.fictional-cdn.com`, entonces cuando se hace una solicitud a `mycdnsubdomain.fictionalcdn.com/assets/smile.png`, el CDN consultará su servidor una vez en `example.com/assets/smile.png` y almacenará la solicitud en caché. La siguiente solicitud al CDN que llega a la misma URL afectará a la copia almacenada en caché. Cuando el CDN puede servir un recurso directamente, la solicitud nunca toca el servidor de Rails. Dado que los recursos de un CDN están geográficamente más cercanos al navegador, la solicitud es más rápida y, dado que el servidor no necesita dedicar tiempo a servir recursos, puede centrarse en servir el código de la aplicación lo más rápido posible.

#### 4.4.1 Configurar un CDN para servir recursos estáticos

Para configurar su CDN, debe ejecutar su aplicación en producción en Internet en una URL disponible públicamente, por ejemplo `example.com`. A continuación, tendrá que registrarse para un servicio de CDN de un proveedor de alojamiento en la nube. Al hacer esto, necesita configurar el "origen" del CDN para que apunte en su sitio web `example.com`, consulte a su proveedor para obtener documentación sobre cómo configurar el servidor de origen.

El CDN provisto debe darle un subdominio personalizado para su aplicación como `mycdnsubdomain.fictional-cdn.com` \(note que `fictional-cdn.com` no es un proveedor CDN válido en el momento de escribir este documento\). Ahora que ha configurado su servidor CDN, debe indicarle a los navegadores que utilicen su CDN para obtener recursos en lugar de su servidor Rails directamente. Puede hacerlo configurando Rails para configurar su CDN como el host activo en lugar de usar una ruta relativa. Para configurar su host de recursos en Rails, debe configurar `config.action_controller.asset_host` en `config/environments/production.rb`:

```ruby
config.action_controller.asset_host = 'mycdnsubdomain.fictional-cdn.com'
```

> Sólo es necesario proporcionar el "host", este es el subdominio y el dominio raíz, no es necesario especificar un protocolo o "esquema" como `http://` o `https://`. Cuando se solicita una página web, el protocolo en el vínculo a su recurso que se genera coincidirá con la forma en que se accede a la página web de forma predeterminada.

También puede establecer este valor a través de una [variable de entorno](https://en.wikipedia.org/wiki/Environment_variable) para facilitar la ejecución de una copia provisional de su sitio:

```ruby
config.action_controller.asset_host = ENV['CDN_HOST']
```

Nota: Usted tendría que configurar `CDN_HOST` en su servidor a `mycdnsubdomain.fictional-cdn.com` para que esto funcione.

Una vez que haya configurado su servidor y su CDN cuando sirva una página web que tenga un recurso:

```ruby
<%= asset_path('smile.png') %>
```

En lugar de devolver una ruta tal como `/assets/smile.png` \(los resúmenes se dejan fuera para facilitar la lectura\). La URL generada tendrá la ruta completa a su CDN.

```ruby
http://mycdnsubdomain.fictional-cdn.com/assets/smile.png
```

Si el CDN tiene una copia de `smile.png` lo servirá al navegador y su servidor ni siquiera sabe que fue solicitado. Si el CDN no tiene una copia, tratará de encontrarla en el "origen" en `example.com/assets/smile.png` y luego la almacenarla para uso futuro.

Si desea servir sólo algunos recursos de su CDN, puede utilizar la opción personalizada `:host`, su helper de recursos, que sobrescribe el valor establecido en `config.action_controller.asset_host`.

```ruby
<%= asset_path 'image.png', host: 'mycdnsubdomain.fictional-cdn.com' %>
```

#### 4.4.2 Personalizar el comportamiento de caché de CDN

Un CDN funciona almacenando contenido en caché. Si el CDN tiene contenido obsoleto o malo, entonces lo está perjudicando en lugar de ayudar a su aplicación. El propósito de esta sección es describir el comportamiento general del almacenamiento en caché de la mayoría de los CDN, su proveedor específico puede comportarse de manera ligeramente diferente.

##### 4.4.2.1 Almacenamiento en caché de la petición CDN

Mientras que un CDN se describe como bueno para el almacenamiento de recursos en caché, en realidad "cachea" toda la petición. Esto incluye el cuerpo del recurso, así como los encabezados. El más importante es `Cache-Control` que le dice a la CDN \(y a los navegadores web\) cómo almacenar el contenido en caché. Esto significa que si alguien solicita un recurso que no existe `/assets/i-dont-exist.png` y su aplicación de Rails devuelve un 404, entonces su CDN probablemente almacenará en caché la página 404 si está presente un encabezado válido de `Cache-Control`.

##### 4.4.2.2 Depuración de encabezado CDN

Una manera de comprobar los encabezados que se almacenan en caché correctamente en su CDN es mediante `curl`. Puede solicitar los encabezados de su servidor y su CDN para verificar que son los mismos:

```
$ curl -I http://www.example/assets/application-
d0e099e021c95eb0de3615fd1d8c4d83.css
HTTP/1.1 200 OK
Server: Cowboy
Date: Sun, 24 Aug 2014 20:27:50 GMT
Connection: keep-alive
Last-Modified: Thu, 08 May 2014 01:24:14 GMT
Content-Type: text/css
Cache-Control: public, max-age=2592000
Content-Length: 126560
Via: 1.1 vegur
```

Versus la copia CDN.

```
$ curl -I http://mycdnsubdomain.fictional-cdn.com/application-
d0e099e021c95eb0de3615fd1d8c4d83.css
HTTP/1.1 200 OK Server: Cowboy Last-
Modified: Thu, 08 May 2014 01:24:14 GMT Content-Type: text/css
Cache-Control:
public, max-age=2592000
Via: 1.1 vegur
Content-Length: 126560
Accept-Ranges:
bytes
Date: Sun, 24 Aug 2014 20:28:45 GMT
Via: 1.1 varnish
Age: 885814
Connection: keep-alive
X-Served-By: cache-dfw1828-DFW
X-Cache: HIT
X-Cache-Hits:
68
X-Timer: S1408912125.211638212,VS0,VE0
```

Compruebe su documentación del CDN para cualquier información adicional que pueden proporcionar como `X-Cache` o para cualquier encabezado adicional que puede agregar.

##### 4.4.2.3 CDNs y el encabezado de control de caché

El encabezado de control es una especificación de la W3C que describe cómo una solicitud se puede almacenar en caché. Cuando no se utiliza n CDN, un navegador utilizará esta información para almacenar en caché el contenido. Esto es muy útil para los recursos que no se modifican y que un navegador no necesita volver a descargar el CSS de un sitio web o JavaScript en cada solicitud. Generalmente queremos que nuestro servidor de Rails diga a nuestro CDN \(y navegador\) que el recurso es "público", lo que significa que cualquier caché puede almacenar la solicitud. También comúnmente queremos establecer `max-age`, que es la cantidad de tiempo que el caché almacenará el objeto antes de invalidar el caché. El valor `max-age` se establece en segundos con un valor máximo posible de `31536000` que es de un año. Puede hacerlo en su aplicación Rails estableciendo

```ruby
config.public_file_server.headers = {
  'Cache-Control' => 'public, max-age=31536000'
}
```

Ahora, cuando su aplicación sirva un recurso en producción, el CDN almacenará el recurso por un año. Dado que la mayoría de los CDN también almacenan en caché los encabezados de la solicitud, este `Cache-Control `será pasado a todos los navegadores futuros que busquen este recurso, el navegador entonces sabe que puede almacenar este recurso durante mucho tiempo antes de necesitarlo de nuevo.

##### 4.4.2.4 CDN e Invalidación de caché basada en URL

La mayoría de los CDN almacenarán en caché el contenido de un recurso en función de la URL completa. Esto significa que una solicitud a

```
http://mycdnsubdomain.fictional-cdn.com/assets/smile-123.png
```

Será una caché completamente diferente de

```
http://mycdnsubdomain.fictional-cdn.com/assets/smile.png
```

Si desea establecer un futuro `max-age` en su `Cache-Control `\(y lo hace\), asegúrese de que cuando cambia sus recursos, su caché se invalida. Por ejemplo, al cambiar la cara sonriente de una imagen de amarillo a azul, desea que todos los visitantes de su sitio obtengan la nueva cara azul. Cuando se utiliza un CDN con el canal de recursos de Rails `config.assets.digest `se establece en `true` por defecto para que cada elemento tenga un nombre de archivo diferente cuando se cambie. De esta manera no tendrá que invalidar manualmente ningún elemento de su caché. Al utilizar un nombre de recurso único y diferente, sus usuarios obtienen el último recurso.







