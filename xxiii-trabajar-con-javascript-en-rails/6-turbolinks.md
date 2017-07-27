# 6- Turbolinks

Rails se suministra con la biblioteca [Turbolinks](https://github.com/turbolinks/turbolinks), que utiliza Ajax para acelerar el procesamiento de páginas en la mayoría de las aplicaciones.

### 6.1 Cómo funciona Turbolinks

Turbolinks adjunta un handler de clics a todos los `<a>` de la página. Si su navegador admite `PushState`, Turbolinks realizará una solicitud Ajax para la página, analizará la respuesta y reemplazará todo el `<body>` de la página con el `<body>` de la respuesta. A continuación, utilizará `PushState` para cambiar la URL  correcta, preservando la semántica de actualización y darle la URL bonita.

Lo único que tienes que hacer para activar Turbolinks es tenerlo en tu Gemfile, y poner `//= require turbolinks` en tu manifiesto de JavaScript, que usualmente es `app/assets/javascripts/application.js`.

Si desea desactivar Turbolinks para ciertos enlaces, agregue un atributo `data-turbolinks="false"` a la etiqueta:

```html
<a href="..." data-turbolinks="false">No turbolinks here</a>.
```

### 6.2 Eventos de cambios de página

Al escribir CoffeeScript, a menudo desea realizar algún tipo de procesamiento tras la carga de la página. Con jQuery, escribirías algo como esto:

```js
$(document).ready ->
  alert "page has loaded!"
```

Sin embargo, como Turbolinks anula el proceso normal de carga de página, el evento en el que se basa no se disparará. Si tiene un código similar, debe cambiar el código para hacerlo:

```js
$(document).on "turbolinks:load", ->
  alert "page has loaded!"
```

Para más detalles, incluyendo otros eventos a los que puedes enlazar, consulta el README de [Turbolinks](https://github.com/turbolinks/turbolinks/blob/master/README.md).

