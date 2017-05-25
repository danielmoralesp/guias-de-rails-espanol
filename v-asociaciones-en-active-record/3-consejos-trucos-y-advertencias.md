### 3.  Consejos, Trucos y Advertencias

Estas son algunas de las cosas que debe saber para hacer un uso eficiente de las asociaciones Active Record en sus aplicaciones de Rails:

* Control del almacenamiento en caché 
* Evitar colisiones de nombres 
* Actualización del esquema 
* Controlar el alcance de la asociación 
* Asociaciones bidireccionales



### 3.1 Control del almacenamiento en caché

Todos los métodos de asociación se construyen alrededor del almacenamiento en caché, lo que mantiene el resultado de la consulta más reciente disponible para operaciones adicionales. La caché se comparte incluso entre los métodos. Por ejemplo:

```ruby
author.books                 # recupera los libros de la base de datos
author.books.size            # utiliza la copia en caché de libros
author.books.empty?          # utiliza la copia en caché de libros
```

Pero ¿qué pasa si desea volver a cargar la caché, porque los datos podrían haber sido cambiados por alguna otra parte de la aplicación? Simplemente llame a "reload" en la asociación:

```ruby
author.books                 # recupera los libros de la base de datos
author.books.size            # utiliza la copia en caché de libros
author.books.reload.empty?   # Descarta la copia en caché de libros
                              # Y vuelve a la base de datos
```



### 3.2 Evitar las colisiones de nombres

No eres libre de usar cualquier nombre para tus asociaciones. Debido a que crear una asociación agrega un método con ese nombre al modelo, es una mala idea darle a una asociación un nombre que ya está utilizado para un método de instancia de `ActiveRecord::Base`. El método de asociación invalidaría el método base y rompería las cosas. Por ejemplo, `attributes` o `connection` son mal nombres para las asociaciones.



