# 3- Templates, Partials y Layouts

Como se mencionó, la salida `HTML` final es una composición de tres elementos Rails: Templates, Partials y Layouts. A continuación se presenta un breve resumen de cada uno de ellos.



### 3.1 Templates

Las plantillas de Action View se pueden escribir de varias maneras. Si el archivo de plantilla tiene una extensión `.erb`, utiliza una mezcla de `ERB` \(Embedded Ruby\) y `HTML`. Si el archivo de plantilla tiene una extensión `.builder`, se utiliza la biblioteca `Builder::XmlMarkup`.

Rails soporta múltiples sistemas de plantilla y utiliza una extensión de archivo para distinguir entre ellos. Por ejemplo, un archivo `HTML` que utiliza el sistema de plantillas `ERB` tendrá `.html.erb` como una extensión de archivo.

#### 3.1.1 ERB

Dentro de una plantilla de `ERB`, el código Ruby se puede incluir usando las etiquetas `<% %>` y `<%= %>`. Las etiquetas `<% %>` se utilizan para ejecutar código Ruby que no devuelven nada, como condiciones, bucles o bloques, y las etiquetas `<%= %>` se utilizan cuando se desea que se devuelva un resultado.

Considere el bucle siguiente para los nombres:

```ruby
<h1>Names of all the people</h1>
<% @people.each do |person| %>
  Name: <%= person.name %><br>
<% end %>
```

El bucle se configura utilizando etiquetas de inserción \(`<% %>`\) y el nombre se inserta utilizando las etiquetas embebidas de salida `(<%= %>`\). Tenga en cuenta que esto no es solo una sugerencia de uso: las funciones de salida regulares como `print` y `puts` no se renderizarán a la vista con plantillas `ERB`. Así que esto estaría mal:

```ruby
<%# WRONG %>
Hi, Mr. <% puts "Frodo" %>
```

Para suprimir los espacios en blanco iniciales y remotos, puede utilizar `<%-  -%>` de forma intercambiable con `<%` y `%>`.



