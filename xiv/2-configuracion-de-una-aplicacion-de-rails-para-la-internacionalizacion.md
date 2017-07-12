# 2- Configuración de una aplicación de Rails para la internacionalización

Hay algunos pasos para iniciar y ejecutar con el soporte para I18n para una aplicación de Rails.

### 2.1 Configurar el módulo I18n

Siguiendo la filosofía de convención sobre configuración, Rails I18n proporciona cadenas de traducción por defecto que son razonables. Cuando se necesitan cadenas de traducción diferentes, pueden anularse.

Rails añade todos los archivos `.rb` y `.yml` desde el directorio `config/locales` a la ruta de carga de las traducciones, automáticamente.

La configuración regional predeterminada de `en.yml` en este directorio que contiene un par de cadenas de traducción:

```ruby
en:
  hello: "Hello world"
```

Esto significa, que en el locale `:en` , la clave `hello` se asignará a la cadena Hello world. Cada cadena dentro de Rails se internacionaliza de esta forma, consulte, por ejemplo, los mensajes de validación de Active Model en el archivo [activemodel/lib/active\_model/locale/en.yml](https://github.com/rails/rails/blob/master/activemodel/lib/active_model/locale/en.yml) o los formatos de hora y fecha en el archivo [activesupport/lib/active\_support/locale/en.yml](https://github.com/rails/rails/blob/master/activesupport/lib/active_support/locale/en.yml) . Puede usar YAML o el Ruby Hashes estándar para almacenar las traducciones en el backend predeterminado \(Simple\).

La biblioteca I18n utilizará el inglés como configuración regional predeterminada, es decir, si no se ha establecido una configuración regional diferente, `:es`se utilizará para buscar traducciones.

> La biblioteca i18n adopta un enfoque pragmático de las claves locales \(después de un debate\), incluyendo sólo la parte local \("idioma"\), como `:en`, `:pl`, no la parte de la región, como `:en-US` o `:en-GB`, Que tradicionalmente se utilizan para separar "lenguas" y "configuración regional" o "dialectos". Muchas aplicaciones internacionales usan sólo el elemento "idioma" de una localidad como `:cs`, `:th` o `:es` \(para checo, tailandés y español\). Sin embargo, también hay diferencias regiones dentro de diferentes grupos lingüísticos que pueden ser importantes. Por ejemplo, en el escenario `:en-US` tendría $ como símbolo de moneda, mientras que en `:es-GB`, tendría £. Nada le impide separar la configuración regional y de otra manera de esta manera: sólo tiene que proporcionar la configuración completa "English - United Kingdom" en el diccionario `:en-GB`. Algunas gemas como Globalize3 pueden ayudar a implementarlo.

La ruta de carga de las traducciones \(`I18n.load_path`\) es una matriz de rutas de acceso a los archivos que se cargarán automáticamente. La configuración de esta ruta permite la personalización de la estructura de directorios de traducciones y el esquema de nomenclatura de archivos.

> El backend tiene una carga perezosa \(lazy-load\) de estas traducciones cuando se busca una traducción por primera vez. Este backend puede intercambiarse con otra cosa incluso después de que las traducciones ya se han anunciado.

Puede cambiar la configuración regional predeterminada, así como configurar las rutas de carga de las traducciones en `config/application.rb` de la siguiente manera:

```ruby
config.i18n.load_path += Dir[Rails.root.join('my', 'locales', '*.{rb,yml}').to_s]
config.i18n.default_locale = :de
```

Se debe especificar la ruta de carga antes de buscar las traducciones. Para cambiar la configuración regional predeterminada de un inicializador, se hace en `config/application.rb`:

```ruby
# config/initializers/locale.rb

# Where the I18n library should search for translation files
I18n.load_path += Dir[Rails.root.join('lib', 'locale', '*.{rb,yml}')]

# Whitelist locales available for the application
I18n.available_locales = [:en, :pt]

# Set default locale to something other than :en
I18n.default_locale = :pt
```

### 2.2 Gestión de la configuración regional a través de las solicitudes \(request\)

La configuración regional predeterminada se utiliza para todas las traducciones a menos que en el `I18n.locale` se establezca explícitamente.

Es probable que una aplicación localizada necesite proporcionar soporte para varios entornos locales. Para lograr esto, la configuración regional se debe establecer al principio de cada solicitud para que todas las cadenas se conviertan utilizando la configuración regional deseada durante la duración de dicha solicitud.

La configuración regional se puede establecer en un `before_action` en el `ApplicationController`:

```ruby
before_action :set_locale

def set_locale
  I18n.locale = params[:locale] || I18n.default_locale
end
```

Este ejemplo ilustra esto utilizando un parámetro de consulta URL para establecer el entorno local \(por ejemplo, `http://example.com/books?locale=pt`\). Con este enfoque, `http://localhost:3000?locale=pt`  renderiza la localización en portugués, mientras que [http://localhost:3000?locale=pde](http://localhost:3000?locale=pde) carga una localización alemana.

La configuración regional se puede establecer utilizando uno de los diferentes enfoques.

#### 2.2.1 Configuración regional del nombre de dominio

Una opción que tiene es establecer la configuración regional desde el nombre de dominio donde se ejecuta la aplicación. Por ejemplo, queremos que `www.example.com` cargue el idioma inglés \(o predeterminado\) y `www.example.es` para cargar el idioma español. Por lo tanto, el nombre de dominio de nivel superior se utiliza para la configuración regional. Esto tiene varias ventajas:

* La configuración regional es una parte obvia de la URL.
* La gente entiende intuitivamente en qué idioma se mostrará el contenido.
* Es muy trivial implementar en Rails.
* Los motores de búsqueda entenderán que el contenido en diferentes idiomas vive en diferentes dominios interrelacionados.

Puede implementarlo así en su `ApplicationController`:

```ruby
before_action :set_locale

def set_locale
  I18n.locale = extract_locale_from_tld || I18n.default_locale
end

# Get locale from top-level domain or return +nil+ if such locale is not available
# You have to put something like:
#   127.0.0.1 application.com
#   127.0.0.1 application.it
#   127.0.0.1 application.pl
# in your /etc/hosts file to try this out locally
def extract_locale_from_tld
  parsed_locale = request.host.split('.').last
  I18n.available_locales.map(&:to_s).include?(parsed_locale) ? parsed_locale : nil
end
```

También podemos establecer la configuración regional desde el subdominio de una manera muy similar:

```ruby
# Get locale code from request subdomain (like http://it.application.local:3000)
# You have to put something like:
#   127.0.0.1 gr.application.local
# in your /etc/hosts file to try this out locally
def extract_locale_from_subdomain
  parsed_locale = request.subdomains.first
  I18n.available_locales.map(&:to_s).include?(parsed_locale) ? parsed_locale : nil
end
```

Si su aplicación incluye un menú para el cambio del `locale`, entonces tendría que hacer algo como esto en el:

```ruby
link_to("Deutsch", "#{APP_CONFIG[:deutsch_website_url]}#{request.env['PATH_INFO']}")
```

Asumiendo que usted fijaría `APP_CONFIG[:deutsch_website_url]` a un cierto valor como `http://www.application.de`.

Esta solución tiene las ventajas mencionadas, sin embargo, es posible que no pueda o no desee proporcionar diferentes localizaciones \("versiones de idioma"\) en diferentes dominios. La solución más obvia sería incluir el código de configuración regional en los parámetros de URL \(o ruta de petición\).

#### 2.2.2 Configuración regional de los parámetros de URL

La forma más habitual de establecer \(y pasar\) el entorno local sería incluirlo en parámetros de URL, como lo hicimos en `I18n.locale = params[:locale]` `before_action` en el primer ejemplo. Nos gustaría tener URLs como `www.example.com/books?locale=ja` o `www.example.com/ja/books` en este caso.

Este enfoque tiene casi el mismo conjunto de ventajas que el establecimiento de la configuración regional del nombre de dominio: a saber, que es `RESTful` y de acuerdo con el resto de la World Wide Web. Sin embargo, requiere un poco más de trabajo para implementar.

Obtener el `locale` de los parámetros y configurarlo en consecuencia no es difícil; Incluyendo en cada URL y por lo tanto pasarlo a través de las solicitudes. Para incluir una opción explícita en cada `URL`, por ejemplo `link_to(books_url(locale: I18n.locale))`, sería tedioso y probablemente imposible, por supuesto.

Rails contiene una infraestructura para "centralizar decisiones dinámicas sobre las URLs" en su `ApplicationController#default_url_options`, lo cual es útil precisamente en este escenario: nos permite establecer "defaults" para `url_for` y los métodos auxiliares dependientes de él \(mediante la implementación o sobre-escritura  de `default_url_options`\).

Podemos incluir algo como esto en nuestro `ApplicationController` entonces:

```ruby
# app/controllers/application_controller.rb
def default_url_options
  { locale: I18n.locale }
end
```

Cada método helper dependiente de `url_for` \(por ejemplo, ayudantes de rutas de nombre, como `root_path` o `root_url`, rutas de resources como `books_path` o `books_url`, etc.\) ahora incluirán automáticamente la configuración regional en la cadena de consulta, de la siguiente manera: `http://localhost:3001?locale=ja`.

Usted puede estar satisfecho con esto. Sin embargo, afecta a la legibilidad de las URL cuando la configuración regional se "pone" al final de cada URL de la aplicación. Por otra parte, desde el punto de vista arquitectónico, los locales suelen estar jerárquicamente por encima de las otras partes del dominio de la aplicación y las URL deben reflejar esto.

Es probable que desee que las URL se vean así: `http://www.example.com/en/books` \(que carga la configuración regional en inglés\) y `http://www.example.com/nl/books` \(que carga la configuración regional holandesa\) . Esto es posible con la estrategia de "over-riding `default_url_options`" desde arriba: solo tienes que configurar tus rutas con el scope de aplicación:

```ruby
# config/routes.rb
scope "/:locale" do
  resources :books
end
```

Ahora, cuando llame al método `books_path`, debe obtener "`/en/books`" \(para la configuración regional predeterminada\). Una URL como `http://localhost:3001/nl/books` debe cargar la configuración regional holandesa, y las siguientes llamadas a `books_path` deben devolver "`/nl/books`" \(porque cambió la configuración regional\).

> Dado que el valor devuelto de `default_url_options` se almacena en caché por solicitud, no se pueden generar URLs en un selector de localización invocando ayudantes en un bucle que se establezca el `I18n.locale` correspondiente en cada iteración. En su lugar, deje el  `I18n.locale` sin tocar y pase una opción explícita `:locale` al ayudante o modifique `request.original_fullpath`.

Si no desea forzar el uso de una configuración regional en sus rutas, puede utilizar un ámbito de ruta opcional \(indicado por los paréntesis\) de la siguiente manera:

```ruby
# config/routes.rb
scope "(:locale)", locale: /en|nl/ do
  resources :books
end
```

Con este enfoque no obtendrá un error de enrutamiento cuando acceda a sus recursos como `http://localhost:3001/books` sin una configuración regional. Esto es útil si desea utilizar la configuración regional predeterminada cuando no se especifica.

Por supuesto, debe tener especial cuidado con la URL raíz \(generalmente "homepage" o "dashboar"\) de su aplicación. Una URL como `http://localhost:3001/nl` no funcionará automáticamente, porque la raíz: "`books#index`" en la declaración de `routes.rb` no toma el locale en cuenta. \(Y con razón: sólo hay una URL "`root`"\).

Probablemente necesitará asignar URL como estas:

```ruby
# config/routes.rb
get '/:locale' => 'dashboard#index'
```

Tenga especial cuidado con el orden de sus rutas, por lo que esta declaración de ruta no debe "comer" a los otros. \(Es posible que desee añadirlo directamente antes de la reclaración a `root`\)

> Echa un vistazo a varias gemas que simplifican el trabajo con rutas: [routing\_filter](https://github.com/svenfuchs/routing-filter/tree/master), [rails-translate-routes](https://github.com/francesc/rails-translate-routes), [route\_translator](https://github.com/enriclluelles/route_translator).

#### 2.2.3 Configuración regional de las preferencias de usuario

Una aplicación con usuarios autenticados puede permitir a los usuarios establecer una preferencia de configuración regional a través de la interfaz de la aplicación. Con este enfoque, la preferencia de configuración de usuario seleccionada del usuario se mantiene en la base de datos y se utiliza para establecer la configuración regional para solicitudes autenticadas por ese usuario.

```ruby
def set_locale
  I18n.locale = current_user.try(:locale) || I18n.default_locale
end
```

#### 2.2.4 Elección de una ubicación implícita

Cuando no se ha establecido una configuración regional explícita para una solicitud \(por ejemplo, mediante uno de los métodos anteriores\), una aplicación debe intentar inferir la configuración regional deseada.

##### 2.2.4.1 Inferir la configuración regional desde el encabezado del idioma

El encabezado `HTTP` de `Accept-Language` indica el idioma preferido para la respuesta de la solicitud. Los navegadores establecen este valor de encabezado en función de la configuración de preferencias de idioma del usuario, lo que lo convierte en una buena primera opción al inferir una configuración regional.

Una implementación trivial de usar un encabezado `Accept-Language` sería:

```ruby
def set_locale
  logger.debug "* Accept-Language: #{request.env['HTTP_ACCEPT_LANGUAGE']}"
  I18n.locale = extract_locale_from_accept_language_header
  logger.debug "* Locale set to '#{I18n.locale}'"
end
 
private
  def extract_locale_from_accept_language_header
    request.env['HTTP_ACCEPT_LANGUAGE'].scan(/^[a-z]{2}/).first
  end
```

En la práctica, un código más robusto es necesario para hacer esto de forma fiable. La biblioteca [http\_accept\_language](https://github.com/iain/http_accept_language/tree/master) de Iain Hecker o el middleware de rack [locale](https://github.com/rack/rack-contrib/blob/master/lib/rack/contrib/locale.rb) de Ryan Tomayko proporcionan soluciones a este problema.

##### 2.2.4.2 Inferir la localización de la geolocalización de la IP

La dirección IP del cliente que realiza la solicitud se puede utilizar para inferir la región del cliente y, por tanto, su entorno local. Servicios como [GeoIP Lite Country](http://dev.maxmind.com/geoip/legacy/geolite/) o gemas como [geocoder](https://github.com/alexreisner/geocoder) pueden ser utilizados para implementar este enfoque.

En general, este enfoque es mucho menos fiable que el uso del encabezado de idioma y no se recomienda para la mayoría de las aplicaciones web.

#### 2.2.5 Almacenamiento de la configuración regional en la sesión o en las cookies

> Puede estar tentado a almacenar el escenario elegido en una sesión o una cookie. Sin embargo, no lo hagas. La configuración regional debe ser transparente y una parte de la URL. De esta manera no romperás las suposiciones básicas de la gente sobre la propia web: si envías una URL a un amigo, deberían ver la misma página y el mismo contenido que tú. Una palabra de fantasía para esto sería que estás siendo `RESTful`. Lea más sobre el enfoque RESTful en los artículos de Stefan Tilkov. A veces hay excepciones a esta regla y se discuten a continuación



