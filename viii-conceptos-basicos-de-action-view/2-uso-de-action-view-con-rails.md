# 2- Uso de Action View con Rails

Para cada controlador hay un directorio asociado en el directorio `app / views` que contiene los archivos de plantilla que forman las vistas asociadas con ese controlador. Estos archivos se utilizan para mostrar la vista que resulta de cada acción del controlador.

```ruby
$ bin/rails generate scaffold article
      [...]
      invoke  scaffold_controller
      create    app/controllers/articles_controller.rb
      invoke    erb
      create      app/views/articles
      create      app/views/articles/index.html.erb
      create      app/views/articles/edit.html.erb
      create      app/views/articles/show.html.erb
      create      app/views/articles/new.html.erb
      create      app/views/articles/_form.html.erb
      [...]
```

Existe una convención de naming para las vistas en Rails. Normalmente, las vistas comparten su nombre con la acción del controlador asociado, como se puede ver más arriba. Por ejemplo, la acción del controlador de `index` del `articles_controller.rb` utilizará el archivo de vista `index.html.erb` en el directorio `app / views / articles`. El `HTML` completo devuelto al cliente se compone de una combinación de este archivo `ERB`, un template de diseño que lo envuelve y todos los partials al que la vista puede hacer referencia. Dentro de esta guía encontrará una documentación más detallada sobre cada uno de estos tres componentes.

