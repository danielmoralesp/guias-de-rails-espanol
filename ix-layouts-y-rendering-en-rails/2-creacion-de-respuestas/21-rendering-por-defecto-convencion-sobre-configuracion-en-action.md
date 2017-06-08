# 2.1 Rendering por defecto: Convención sobre configuración en Action

Has escuchado que Rails promueve la "convención sobre la configuración". El rendering por defecto es un ejemplo excelente de esto. De forma predeterminada, los controladores de Rails automáticamente reproducen vistas con nombres que corresponden a rutas válidas. Por ejemplo, si tiene este código en su clase `BooksController`:

```ruby
class BooksController < ApplicationController
end
```

Y lo siguiente en tu archivo de rutas:

```ruby
resources :books
```

Y usted tiene un archivo de vistas `app/views/books/index.html.erb`:

```ruby
<h1>Books are coming soon!</h1>
```

Rails renderizará automáticamente `app/views/books/index.html.erb` cuando navegue a `/books` y verá "Books are coming soon!" En la pantalla.

Sin embargo, este pantallazo es poco útil, por lo que pronto deberá crear su modelo de `book` y agregará la acción de `index` de `BooksController`:

```ruby
class BooksController < ApplicationController
  def index
    @books = Book.all
  end
end
```

Tenga en cuenta que no tenemos que llamar a `render` de forma explicita al final de la acción `index` de acuerdo con el principio de "convención sobre configuración". La regla es que si no se hace explícitamente algo al final de una acción del controlador, Rails buscará automáticamente la plantilla `action_name.html.erb` en la ruta de vista del controlador y la procesará. En este caso, Rails procesará el archivo `app/views/books/index.html.erb`.

Si queremos mostrar las propiedades de todos los libros en nuestra vista, podemos hacerlo con una plantilla `ERB` como esta:

```ruby
<h1>Listing Books</h1>
 
<table>
  <tr>
    <th>Title</th>
    <th>Summary</th>
    <th></th>
    <th></th>
    <th></th>
  </tr>
 
<% @books.each do |book| %>
  <tr>
    <td><%= book.title %></td>
    <td><%= book.content %></td>
    <td><%= link_to "Show", book %></td>
    <td><%= link_to "Edit", edit_book_path(book) %></td>
    <td><%= link_to "Remove", book, method: :delete, data: { confirm: "Are you sure?" } %></td>
  </tr>
<% end %>
</table>
 
<br>
 
<%= link_to "New book", new_book_path %>
```

El rendering actual se realiza mediante subclases de `ActionView::TemplateHandlers`. Esta guía no entra en ese proceso, pero es importante saber que la extensión de archivo en su vista controla la elección del controlador de plantilla. Comenzando con Rails 2, las extensiones estándar son `.erb` para `ERB` \(`HTML` con Ruby incrustado \) y `.builder` para Builder \(generador de XML\).

