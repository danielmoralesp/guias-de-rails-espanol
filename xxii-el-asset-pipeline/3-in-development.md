# 3- In Development

En el modo de desarrollo, los recursos se sirven como archivos separados en el orden en que se especifican en el archivo de manifiesto.

Este manifiesto `app/assets/javascripts/application.js`:

```js
//= require core
//= require projects
//= require tickets
```

Generaría este HTML:

```html
<script src="/assets/core.js?body=1"></script>
<script src="/assets/projects.js?body=1"></script>
<script src="/assets/tickets.js?body=1"></script>
```

El param `body` es requerido por Sprockets.

### 3.1 Comprobación de errores en tiempo de ejecución

De forma predeterminada, la canalización de recursos comprueba los posibles errores en el modo de desarrollo durante el tiempo de ejecución. Para desactivar este comportamiento puede establecer:

```ruby
config.assets.raise_runtime_errors = false
```

Cuando esta opción es `true`, la canalización de los recursos comprobará si todos los recursos cargados en su aplicación están incluidos en la lista `config.assets.precompile`. Si `config.assets.digest` también es `true`, la canalización de recursos requerirá que todas las solicitudes de recursos incluyan digests \(resúmenes\).

### 3.2 Levantar un error cuando un recurso no se encuentra

Si está utilizando `sprockets-rails >= 3.2.0 `puede configurar lo que sucede cuando se realiza una búsqueda de recursos y no se encuentra nada. Si desactiva el "asset fallback", se generará un error cuando no se pueda encontrar un activo.

```ruby
config.assets.unknown_asset_fallback = false
```

Si "asset fallback" esta habilitado, cuando no se pueda encontrar un recursos, se mostrará la ruta de acceso y no se producirá ningún error. El comportamiento ante un fallo de recursos está habilitado de forma predeterminada.

### 3.3 Cómo desactivar los digest

Puede desactivar los digest mediante la actualización de `config/environments/development.rb` para incluir:

```ruby
config.assets.digest = false
```

Cuando esta opción es `true`, se generarán los digest para las URL de los recursos.

### 3.4 Como desactivar el debugging

Puede desactivar el modo de debugging debe incluir en `config/environments/development.rb` lo siguiente:

```ruby
config.assets.debug = false
```

Cuando el modo de depuración está desactivado, Sprockets concatena y ejecuta los preprocesadores necesarios en todos los archivos. Con el modo de depuración desactivado, el manifiesto generaría en su lugar:

```js
<script src="/assets/application.js"></script>
```

Los recursos se compilan y se almacenan en caché en la primera solicitud después de iniciarse el servidor. Sprockets establece una cabecera HTTP de Cache-Control que se debe revalidar para reducir la sobrecarga de la petición en las solicitudes posteriores. En estas, el navegador obtiene una respuesta 304 \(no modificada\).

Si alguno de los archivos del manifiesto ha cambiado entre las solicitudes, el servidor responde con un nuevo archivo compilado.

El modo de depuración también se puede habilitar en los métodos de ayuda de Rails:

```ruby
<%= stylesheet_link_tag "application", debug: true %>
<%= javascript_include_tag "application", debug: true %>
```

La opción `:debug` es redundante si el modo de depuración ya está activado.

También puede habilitar la compresión en modo de desarrollo como comprobación de sanidad y deshabilitarla a petición según sea necesario para la depuración.







