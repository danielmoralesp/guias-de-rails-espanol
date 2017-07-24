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





