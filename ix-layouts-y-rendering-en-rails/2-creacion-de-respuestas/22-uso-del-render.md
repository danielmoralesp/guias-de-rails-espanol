# 2.2 Uso del render

En la mayoría de los casos, el método `ActionController::Base#render` hace el trabajo pesado de renderizar el contenido de su aplicación para su uso en el navegador. Hay una variedad de maneras de personalizar el comportamiento de `render`. Puede renderizar la vista predeterminada para una plantilla de Rails, o una plantilla específica, o un archivo, o código inline, o nada en absoluto. Puede procesar texto, `JSON` o `XML`. También puede especificar el tipo de contenido o el estado `HTTP` de la respuesta renderizada.

Si desea ver los resultados exactos de una llamada a `render` sin necesidad de inspeccionarla en un navegador, puede llamar a `render_to_string`. Este método toma exactamente las mismas opciones que `render`, pero devuelve una cadena en lugar de enviar una respuesta al navegador.

### 2.2.1 Visualización de una acción

Si desea representar la vista que corresponde a una plantilla diferente dentro del mismo controlador, puede utilizar `render` con el nombre de la vista:

```ruby
def update
  @book = Book.find(params[:id])
  if @book.update(book_params)
    redirect_to(@book)
  else
    render "edit"
  end
end
```

Si falla la llamada a `update`, llama nuevamente a la acción `update` y este controlador mostrará la plantilla `edit.html.erb` perteneciente al mismo controlador.

Si lo prefiere, puede utilizar un símbolo en lugar de una cadena para especificar la acción a renderizar:

```ruby
def update
  @book = Book.find(params[:id])
  if @book.update(book_params)
    redirect_to(@book)
  else
    render :edit
  end
end
```

### 2.2.2 Procesamiento de un template de una acción desde otro controlador

¿Qué sucede si desea procesar una plantilla desde un controlador completamente distinto del que contiene el código de acción? También puede hacer eso con `render`, que acepta la ruta completa \(relativa a `app/views`\) de la plantilla a renderizar. Por ejemplo, si ejecuta código en un `AdminProductsController` que vive en `app/controllers/admin`, puede renderizar los resultados de una acción a una plantilla en `app/views/products` de esta manera:

```ruby
render "products/show"
```

Rails sabe que esta vista pertenece a un controlador diferente debido all carácter de barra diagonal incrustado en la cadena. Si desea ser explícito, puede utilizar la opción `:template` \(que se requería en Rails 2.2 y anteriores\):

```ruby
render template: "products/show"
```

### 2.2.3 Procesamiento de un archivo arbitrario

El método `render` también puede utilizar una vista que está totalmente fuera de la aplicación:

```ruby
render file: "/u/apps/warehouse_app/current/app/views/products/show"
```

La opción :file toma una ruta absoluta del sistema de archivos. Por supuesto, necesita tener derechos sobre la vista que está utilizando para procesar el contenido.

> El uso de la opción :file en combinación con la entrada de los usuarios puede dar lugar a problemas de seguridad ya que un atacante podría utilizar esta acción para acceder a archivos sensibles a la seguridad en su sistema de archivos.
>
> De forma predeterminada, el archivo se procesa utilizando el layout actual.
>
> Si está ejecutando Rails en Microsoft Windows, debería utilizar la opción :file para procesar un archivo, ya que los nombres de archivo de Windows no tienen el mismo formato que los nombres de archivo Unix.

### 2.2.4 Envolviéndolo

Las tres formas de representación anteriores \(renderizar otra plantilla dentro del controlador, renderizar una plantilla dentro de otro controlador y renderizar un archivo arbitrario en el sistema de archivos\) son en realidad variantes de la misma acción.

De hecho, en la clase `BooksController`, dentro de la acción `update` donde queremos renderizar la plantilla `edit` si el `book` no se actualiza correctamente, todas las siguientes llamadas de renderizarían se harán a la plantilla `edit.html.erb` que esta en `views/books`

```ruby
render :edit
render action: :edit
render "edit"
render "edit.html.erb"
render action: "edit"
render action: "edit.html.erb"
render "books/edit"
render "books/edit.html.erb"
render template: "books/edit"
render template: "books/edit.html.erb"
render "/path/to/rails/app/views/books/edit"
render "/path/to/rails/app/views/books/edit.html.erb"
render file: "/path/to/rails/app/views/books/edit"
render file: "/path/to/rails/app/views/books/edit.html.erb"
```

Cuál utiliza usted es realmente una cuestión de estilo y de convención, pero la regla de oro es utilizar el más simple y que tiene sentido para el código que usted está escribiendo.

### 

### 2.2.5 Uso de render con :inline

El método `render` puede prescindir de una vista completa, si usa la opción `:inline` para suministrar el `ERB` como parte de la llamada al método. Esto es perfectamente válido:

```ruby
render inline: "<% products.each do |p| %><p><%= p.name %></p><% end %>"
```

Rara vez hay una buena razón para usar esta opción. La mezcla de `ERB` en sus controladores derrota la orientación MVC de Rails y dificultará que otros desarrolladores sigan la lógica de su proyecto. Utilice una vista `erb` separada y en su lugar respectivo.

De forma predeterminada, el rendering inline utiliza `ERB`. Puede forzarlo a que utilice `Builder` en su lugar con la opción `:type`.

```ruby
render inline: "xml.p {'Horrid coding practice!'}", type: :builder
```



### 2.2.6 Renderizado de texto

Puede enviar texto plano \(sin markups\) - de nuevo al navegador usando la opción `:plain` para renderizar:

```ruby
render plain: "OK"
```

> Renderizar texto plano es muy útil cuando respondes a un Ajax o a web services request que esperan algo distinto del `HTML` apropiado.

De forma predeterminada, si utiliza la opción `:plain`, el texto se procesa sin utilizar el layout actual. Si desea que Rails coloque el texto en el layout actual, debe agregar la opción layout `:true` y usar la extensión `.txt.erb` para el archivo de diseño.



### 2.2.7 Renderización de HTML

Puede enviar una cadena `HTML` de vuelta al navegador mediante la opción `:html` para renderizar:

```ruby
render html: "<strong>Not Found</strong>".html_safe
```

> Esto es útil cuando estás procesando un pequeño fragmento de código `HTML`. Sin embargo, es posible que desee considerar moverlo a un archivo template si el marcado es complejo.
>
> Cuando se utiliza la opción `:html`, las entidades HTML se escaparán si la cadena no está marcada como `HTML` seguro mediante el uso del método `html_safe`.







