# 2- Cómo utilizar el canalizador de recursos

En versiones anteriores de Rails, todos los activos se encontraban en subdirectorios `public`, como imágenes, javascripts y hojas de estilo. Con la canalización de recursos, la ubicación preferida para estos recursos es ahora el directorio `app/assets`. Los archivos en este directorio son servidos por el middleware Sprockets.

Los recursos todavía se pueden colocar en la herencia `public`. Cualquier recurso en `public` será servido como recursos estáticos por la aplicación o servidor web cuando `config.public_file_server.enabled` esté establecido en `true`. Debe utilizar `app/assets` para archivos que deben someterse a algún pre-procesamiento antes de que se sirvan.

En la producción, Rails precompila estos archivos en public/assets por defecto. Las copias precompiladas son servidas como recursos estáticos por el servidor web. Los archivos de `app/assets` nunca se sirven directamente en producción.

### 2.1 Recursos específicos del controlador

Cuando genera un scaffold o un controlador, Rails también genera un archivo JavaScript \(o un archivo CoffeeScript si la gema  `coffee-rails` está en el archivo `Gemfile`\) y un archivo de hoja de estilos en cascada \(o archivo SCSS si `sass-rails` está en el archivo Gemfile\) en ese controlador. Además, al generar un scaffold, Rails genera el archivo `scaffolds.css` \(o `scaffolds.scss` si `sass-rails` está en el Gemfile\).

Por ejemplo, si genera un `ProjectsController`, Rails también agregará un nuevo archivo en `app/assets/javascripts/projects.coffee` y otro en `app/assets/stylesheets/projects.scss`. De forma predeterminada, estos archivos estarán listos para ser utilizados por su aplicación inmediatamente usando la directiva `require_tree`. Vea [Manifest Files and Directives](http://guides.rubyonrails.org/asset_pipeline.html#manifest-files-and-directives) para obtener más detalles sobre `require_tree`.

También puede optar por incluir hojas de estilo específicas del controlador y archivos JavaScript sólo en sus respectivos controladores utilizando lo siguiente:

```ruby
<%= javascript_include_tag params[:controller] %>
```

o

```ruby
<%= stylesheet_link_tag params[:controller] %>
```

Al hacer esto, asegúrese de que no está utilizando la directiva `require_tree`, ya que resultará en que sus recursos se incluyan más de una vez.

> Al utilizar la precompilación de activos, deberá asegurarse de que los activos del controlador se precompilan al cargarlos por página. De forma predeterminada, los archivos `.coffee` y .`scss` no se precompilarán por sí solos. Consulte Precompilación de activos para obtener más información sobre cómo funciona la precompilación.
>
> Debe tener un tiempo de ejecución soportado por ExecJS para poder utilizar CoffeeScript. Si está utilizando macOS o Windows, tiene un tiempo de ejecución de JavaScript instalado en su sistema operativo. Consulte la documentación de ExecJS para conocer todos los tiempos de ejecución de JavaScript admitidos.

También puede desactivar la generación de archivos de recursos específicos del controlador agregando lo siguiente a su configuración `config/application.rb`:

```ruby
config.generators do |g|
  g.assets false
end
```

### 2.2 Organización de los recursos

Los recursos de la canalización se pueden colocar dentro de una aplicación en una de tres ubicaciones: `app/assets`, `lib/assets` o `vendor/assets`.

* `app/assets` es para recursos propiedad de la aplicación, como imágenes personalizadas, archivos JavaScript o hojas de estilo.
* `lib/assets` es para el código de sus propias bibliotecas que no encaja realmente en el ámbito de la aplicación o las bibliotecas que se comparten entre aplicaciones.
* `vendor/assets` es para recursos pertenecientes a entidades externas, como código para complementos JavaScript y frameworks CSS. Tenga en cuenta que el código de terceros con referencias a otros archivos también procesados por el componente Pipeline \(imágenes, hojas de estilo, etc.\) tendrá que volver a escribirse para utilizar ayudantes como `asset_path`.

> Si se está actualizando desde Rails 3, tenga en cuenta que los activos de `lib/assets` o `vendor/assets` están disponibles para su inclusión a través de los manifiestos de la aplicación, pero ya no forman parte de la matriz de precompilación. Consulte Precompilación de activos para obtener orientación.

#### 2.2.1 Rutas de búsqueda

Cuando se hace referencia a un archivo desde un manifiesto o un helper, Sprockets busca en las tres ubicaciones de recursos predeterminados para ello.

Las ubicaciones predeterminadas son: los directorios `images`, `javascripts` y `stylesheets` bajo la carpeta `app/assets`, pero estos subdirectorios no son especiales: cualquier ruta de acceso en `assets/*` será buscada.

Por ejemplo, estos archivos:

```ruby
app/assets/javascripts/home.js
lib/assets/javascripts/moovinator.js
vendor/assets/javascripts/slider.js
vendor/assets/somepackage/phonebox.js
```

Será referenciado en un manifiesto como este:

```ruby
//= require home
//= require moovinator
//= require slider
//= require phonebox
```

Los recursos dentro de subdirectorios también se pueden acceder.

```ruby
app/assets/javascripts/sub/something.js
```

es referenciado como:

```ruby
//= require sub/something
```

Puede ver la ruta de búsqueda inspeccionando `Rails.application.config.assets.paths` en la consola de Rails.

Además de los paths `assets/*` estándar, se pueden agregar rutas adicionales \(totalmente calificadas\) a la canalización en `config/application.rb`. Por ejemplo:

```ruby
config.assets.paths << Rails.root.join("lib", "videoplayer", "flash")
```

Las rutas se recorren en el orden en que se producen en la ruta de búsqueda. De forma predeterminada, esto significa que los archivos de `app/assets` tienen prioridad y se enmascaran las rutas correspondientes en `lib` y `vendor`.

> Es importante tener en cuenta que los recursos a los que desea hacer referencia fuera de un manifiesto deben agregarse a la matriz de precompilación o no estarán disponibles en el entorno de producción.

#### 2.2.2 Utilización de archivos index

Sprockets utiliza archivos denominados `index` \(con las extensiones correspondientes\) para un propósito especial.

Por ejemplo, si tiene una biblioteca `jQuery` con muchos módulos, que se almacena en `lib/assets/javascripts/library_name`, el archivo `lib/assets/javascripts/library_name/index.js` sirve como manifiesto para todos los archivos de esta biblioteca. Este archivo podría incluir una lista de todos los archivos necesarios en orden, o una simple directiva `require_tree`.

La biblioteca como un todo se puede acceder en el manifiesto de la aplicación de la siguiente manera:

```
//= require library_name
```

Esto simplifica el mantenimiento y mantiene las cosas limpias permitiendo que el código relacionado se agrupe antes de su inclusión en otro lugar.

### 2.3 Enlaces a los recursos

Sprockets no agrega ningún método nuevo para acceder a tus recursos: aún debes usar el conocido `javascript_include_tag` y `stylesheet_link_tag`:

```ruby
<%= stylesheet_link_tag "application", media: "all" %>
<%= javascript_include_tag "application" %>
```

Si utiliza la gem `turbolinks` , que se incluye de forma predeterminada en Rails, incluya la opción '`data-turbolinks-track`' que hace que los turbolinks comprueben si un recurso se ha actualizado y si lo carga en la página:

```ruby
<%= stylesheet_link_tag "application", media: "all", "data-turbolinks-track" => "reload" %>
<%= javascript_include_tag "application", "data-turbolinks-track" => "reload" %>
```

En vistas regulares, puede acceder a las imágenes del directorio` app/assets/images` de la siguiente manera:

```ruby
<%= image_tag "rails.png" %>
```

Siempre que el canal esté habilitado dentro de su aplicación \(y no deshabilitado en el contexto actual del entorno\), este archivo es servido por Sprockets. Si existe un archivo en `public/assets/rails.png` es servido por el servidor web.

Alternativamente, una solicitud de un archivo con un hash SHA256 como `public/assets/railsf90d8a84c707a8dc923fca1ca1895ae8ed0a09237f6992015fef1e11be77c023.png` se trata de la misma vía. La forma en que se generan estos hashes se trata en la sección in Development más adelante en esta guía.

Los Sprockets también mirarán a través de las rutas especificadas en `config.assets.paths`, que incluye las rutas standard de la aplicación y cualquier ruta añadida por los motores de Rails.

Las imágenes también se pueden organizar en subdirectorios si es necesario y, a continuación, se puede acceder especificando el nombre del directorio en la etiqueta:

```ruby
<%= image_tag "icons/rails.png" %>
```

> Si está precompilando sus recursos \(vea in Production más adelante\), enlazar a un recurso que no existe generará una excepción en la página de llamada. Esto incluye enlazar a una cadena en blanco. Como tal, tenga cuidado con `image_tag` y los otros helpers con los datos suministrados por el usuario.

#### 2.3.1 CSS y ERB

E canalizador de recursos evalúa automáticamente el `ERB`. Esto significa que si agrega una extensión `erb` a un recurso CSS \(por ejemplo, `application.css.erb`\), los helpers como `asset_path` estarán disponibles en sus reglas `CSS`:

```css
.class { background-image: url(<%= asset_path 'image.png' %>) }
```

Esto escribe la ruta del recurso particular al que se está haciendo referencia. En este ejemplo, tendría sentido tener una imagen en una de las rutas de carga de activos, como `app/assets/images/image.png`, que se referenciaría aquí. Si esta imagen ya está disponible en `public/assets` como un archivo de huellas dactilares, se hace referencia a esa ruta.

Si desea utilizar una [URI de datos](https://en.wikipedia.org/wiki/Data_URI_scheme), un método para incrustar los datos de imagen directamente en el archivo CSS, puede utilizar el asistente `asset_data_uri`.

```css
#logo { background: url(<%= asset_data_uri 'logo.png' %>) }
```

Esto inserta una URI de datos correctamente formateados en el CSS fuente .

Tenga en cuenta que la etiqueta de cierre no puede ser del estilo `-%>`.







