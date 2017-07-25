# 7- Agregar recursos a sus gemas

Los recursos también pueden provenir de fuentes externas en forma de gemas.

Un buen ejemplo de esto es la gema `jquery-rails` que viene con Rails como la gema de biblioteca JavaScript estándar. Esta gema contiene una clase de motor que hereda de `Rails::Engine`. Haciendo esto, Rails es informado de que el directorio para esta gema puede contener recursos y los directorios `app/assets`, `lib/assets` y `vendor/assets` de este motor se agregan a la ruta de búsqueda de Sprockets.



