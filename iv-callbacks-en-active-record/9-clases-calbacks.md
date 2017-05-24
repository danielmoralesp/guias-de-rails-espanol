# 9. Clases Callbacks

Algunas veces los métodos callback que escribirás serán lo suficientemente útiles como para ser reutilizados por otros modelos. Active Record hace posible crear clases que encapsulen métodos callbacks, entonces se vuelve muy sencillo reutilizarlos. 

Aquí un ejemplo donde creamos una clase con un callback `after_destroy` para un modelo `PictureFile`:

```ruby
class PictureFileCallbacks
  def after_destroy(picture_file)
    if File.exist?(picture_file.filepath)
      File.delete(picture_file.filepath)
    end
  end
end
```

Cuando declaramos dentro de una clase, como la de arriba, los métodos callback recibirán el objeto del modelo como un parámetro. Ahora podemos utilizar la clase callback en el modelo:

```ruby
class PictureFile < ActiveRecord::Base
  after_destroy PictureFileCallbacks.new
end
```

> Nota que necesitamos instanciar un nuevo objeto `PictureFileCallbacks`, por haber declarado nuestro callback como un método de instancia. Esto es particularmente útil si los callbacks hacen uso del estado del objeto instanciado. Frecuentemente, sin embargo, tendrá más sentido para declarar los callbacks como métodos de clase:



```ruby
class PictureFileCallbacks
  def self.after_destroy(picture_file)
    if File.exist?(picture_file.filepath)
      File.delete(picture_file.filepath)
    end
  end
end
```

Si el método callback es declarado de esta manera, no será necesario instanciar un objeto PictureFileCallbacks.

```ruby
class PictureFile < ActiveRecord::Base
  after_destroy PictureFileCallbacks
end
```

Puedes declarar tantos callbacks como quieras dentro de tu clase callback.

