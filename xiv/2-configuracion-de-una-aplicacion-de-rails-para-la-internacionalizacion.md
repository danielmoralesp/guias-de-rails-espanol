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

La biblioteca I18n utilizará el inglés como configuración regional predeterminada, es decir, si no se ha establecido una configuración regional diferente, `:es `se utilizará para buscar traducciones.

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

Este ejemplo ilustra esto utilizando un parámetro de consulta URL para establecer el entorno local \(por ejemplo, `http://example.com/books?locale=pt`\). Con este enfoque, `http://localhost:3000?locale=pt`  renderiza la localización en portugués, mientras que http://localhost:3000?locale=pde carga una localización alemana.

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



