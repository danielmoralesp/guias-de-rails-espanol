# 2.3 Uso de redirect\_to

Otra forma de manejar las respuestas de retorno a una solicitud `HTTP` es con `redirect_to`. Como se ha visto, `render` le dice a Rails qué vista \(u otro asset\) debe usar para construir una respuesta. El método `redirect_to` hace algo completamente diferente: le dice al navegador que envíe una nueva solicitud para una URL diferente. Por ejemplo, puede redirigir desde donde quiera que esté en su código hasta el `index` de fotos de su aplicación con esta llamada:

```ruby
redirect_to photos_url
```

Puede utilizar `redirect_back` para devolver al usuario a la página de la que proceden. Esta ubicación se extrae del encabezado `HTTP_REFERER` que el navegador no garantiza que se establezca, por lo que debe proporcionar el `fallback_location` para utilizar en este caso.

```ruby
redirect_back(fallback_location: root_path)
```

> `redirect_to` y `redirect_back` no se detienen y retornan inmediatamente desde la ejecución del método, sino que simplemente establecen respuestas `HTTP`. Las declaraciones que ocurran después de ellos en un método serán ejecutadas. Usted puede detener por un `return` explícito o algún otro mecanismo de detención, si es necesario.



### 2.3.1 Obtención de un código de estado de redirección diferente

Rails utiliza el código de estado `HTTP 302`, un redireccionamiento temporal, cuando llama a `redirect_to`. Si desea utilizar un código de estado diferente, tal vez `301`, un redireccionamiento permanente, puede utilizar la opción `:status`:

```ruby
redirect_to photos_path, status: 301
```

Al igual que la opción` :status` para `render`, `:status` para `redirect_to` acepta las designaciones de encabezados numéricos y simbólicos.



### 2.3.2 La diferencia entre render y redirect\_to

A veces, los desarrolladores sin experiencia piensan en `redirect_to` como una especie de comando `go-to`, que mueve la ejecución de un lugar a otro en su código Rails. Esto no es correcto. Su código deja de ejecutarse y espera una nueva solicitud para el navegador. Sucede que le has dicho al navegador qué solicitud debe hacer a continuación, enviando un código de estado `HTTP 302`.

Considere estas acciones para ver la diferencia:

```ruby
def index
  @books = Book.all
end
 
def show
  @book = Book.find_by(id: params[:id])
  if @book.nil?
    render action: "index"
  end
end
```

Con el código de esta forma, probablemente habrá un problema si la variable `@book` es `nil`. Recuerde que una acción `:render action` no ejecuta ningún código en la acción de destino, de modo que nada configurará la variable `@books` que probablemente requiera la vista de `index`. Una manera de arreglar esto es redireccionar en lugar de renderizar:

```ruby
def index
  @books = Book.all
end
 
def show
  @book = Book.find_by(id: params[:id])
  if @book.nil?
    redirect_to action: :index
  end
end
```

Con este código, el navegador hará una nueva solicitud para la página `index`, y el código en el método `index` se ejecutará, y todo estará bien.

El único inconveniente de este código es que requiere un viaje de ida y vuelta al navegador: el navegador solicita la acción `show` con `/books/1` y el controlador detecta que no hay libros, por lo que el controlador envía una respuesta de redirección 302 al navegador diciéndole que vaya a `/books/`, el navegador cumple y envía una nueva solicitud al controlador solicitando ahora la acción `index`, el controlador obtiene todos los libros de la base de datos y procesa la plantilla `index`, devolviéndola a la navegador que luego lo muestra en su pantalla.

Mientras que en una pequeña aplicación, esta latencia añadida podría no ser un problema, es algo en lo que pensar si el tiempo de respuesta es una preocupación. Podemos demostrar una manera de manejar esto con un ejemplo:

```ruby
def index
  @books = Book.all
end
 
def show
  @book = Book.find_by(id: params[:id])
  if @book.nil?
    @books = Book.all
    flash.now[:alert] = "Your book was not found"
    render "index"
  end
end
```

Esto detectaría que no hay libros con el `ID` especificado, llena la variable de instancia de `@books` con todos los libros del modelo y luego renderiza directamente la plantilla `index.html.erb`, devolviéndola al navegador con un mensaje de alerta `flash` para decirle al usuario lo que pasó.











