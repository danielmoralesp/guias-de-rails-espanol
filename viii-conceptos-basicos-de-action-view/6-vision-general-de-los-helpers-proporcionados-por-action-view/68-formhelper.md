# 6.8 FormHelper

Los ayudantes de formularios están diseñados para facilitar el trabajo con modelos en comparación con el uso de elementos `HTML` estándar proporcionando un conjunto de métodos para crear formularios basados ​​en sus modelos. Este ayudante genera `HTML` para los formularios, proporcionando un método para cada tipo de entrada \(por ejemplo, text, password, select, etc.\). Cuando se envía el formulario \(es decir, cuando el usuario pulsa el botón de envío o `form.submit` se envía a través de JavaScript\), las entradas del formulario se incluirán en el objeto `params` y se devolverán al controlador.

Hay dos tipos de ayudantes de formulario: los que trabajan específicamente con los atributos del modelo y los que no. `FormHeper` se ocupa de los que trabajan con atributos del modelo; Para ver un ejemplo de ayudantes de formulario que no funcionan con atributos de modelo, consulte la documentación `ActionView::Helpers::FormTagHelper`.

El método principal de este ayudante, `form_for`, le da la capacidad de crear un formulario para una instancia de modelo; Por ejemplo, digamos que usted tiene un modelo `Person` y desea crear una nueva instancia del mismo:

```ruby
# Nota: se habrá creado una variable @person en el controlador (por ejemplo, @person = Person.new)
<%= form_for @person, url: { action: "create" } do |f| %>
  <%= f.text_field :first_name %>
  <%= f.text_field :last_name %>
  <%= submit_tag 'Create' %>
<% end %>
```

El `HTML` generado para esto sería:

```ruby
<form action="/people/create" method="post">
  <input id="person_first_name" name="person[first_name]" type="text" />
  <input id="person_last_name" name="person[last_name]" type="text" />
  <input name="commit" type="submit" value="Create" />
</form>
```

El objeto `params` creado cuando se envía este formulario se vería así:

```ruby

{ "action" => "create", "controller" => "people", "person" => { "first_name" => "William", "last_name" => "Smith" } }
```

El hash de parámetros tiene un valor de anidado de persona, por lo que se puede acceder con `params[:person]` en el controlador.



### 6.8.1 check\_box

Devuelve una etiqueta de casilla de verificación adaptada para acceder a un atributo especificado.

```ruby
# Digamos que @article.validated? Es 1:
check_box("article", "validated")
# => <input type="checkbox" id="article_validated" name="article[validated]" value="1" />
#    <input name="article[validated]" type="hidden" value="0" />
```



### 6.8.2 fields\_for

Crea un scope alrededor de un objeto de modelo específico como `form_for`, pero no crea las etiquetas de formulario por sí mismo. Esto hace que `fields_for` sea adecuado para especificar objetos de modelo adicionales en el mismo form:

```ruby
<%= form_for @person, url: { action: "update" } do |person_form| %>
  First name: <%= person_form.text_field :first_name %>
  Last name : <%= person_form.text_field :last_name %>
 
  <%= fields_for @person.permission do |permission_fields| %>
    Admin?  : <%= permission_fields.check_box :admin %>
  <% end %>
<% end %>
```













