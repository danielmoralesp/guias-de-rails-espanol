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



### 6.8.3 file\_field

Devuelve una etiqueta de entrada de carga de archivo adaptada para acceder a un atributo especificado.

```ruby
file_field(:user, :avatar)
# => <input type="file" id="user_avatar" name="user[avatar]" />
```



### 6.8.4 form\_for

Crea un formulario y un scope alrededor de un objeto de modelo específico que se utiliza como base para preguntar los valores de los campos.

```ruby
<%= form_for @article do |f| %>
  <%= f.label :title, 'Title' %>:
  <%= f.text_field :title %><br>
  <%= f.label :body, 'Body' %>:
  <%= f.text_area :body %><br>
<% end %>
```

### 

### 6.8.5 hidden\_field

Devuelve una etiqueta de entrada oculta adaptada para acceder a un atributo especificado.

```ruby
hidden_field(:user, :token)
# => <input type="hidden" id="user_token" name="user[token]" value="#{@user.token}" />
```



### 6.8.6 label

Devuelve una etiqueta `label` adaptada para etiquetar un campo de entrada para un atributo especificado.

```ruby
label(:article, :title)
# => <label for="article_title">Title</label>
```



### 6.8.7 password\_field

Devuelve una etiqueta de entrada del tipo "contraseña" adaptada para acceder a un atributo especificado.

```ruby
password_field(:login, :pass)
# => <input type="text" id="login_pass" name="login[pass]" value="#{@login.pass}" />
```



### 6.8.8 radio\_button

Devuelve una etiqueta de radio button para acceder a un atributo especificado.

```ruby
# Digamos que @article.category devuelve "Rails":
radio_button("article", "category", "rails")
radio_button("article", "category", "java")
# => <input type="radio" id="article_category_rails" name="article[category]" value="rails" checked="checked" />
#    <input type="radio" id="article_category_java" name="article[category]" value="java" />
```



### 6.8.9 text\_area

Devuelve un conjunto de etiquetas de apertura y cierre de `textarea` adaptadas para acceder a un atributo especificado.

```ruby
text_area(:comment, :text, size: "20x30")
# => <textarea cols="20" rows="30" id="comment_text" name="comment[text]">
#      #{@comment.text}
#    </textarea>
```



### 6.8.10 text\_field

Devuelve una etiqueta de entrada del tipo "texto" adaptada para acceder a un atributo especificado.

```ruby
text_field(:article, :title)
# => <input type="text" id="article_title" name="article[title]" value="#{@article.title}" />
```



### 6.8.11 email\_field

Devuelve una etiqueta de entrada del tipo "correo electrónico" adaptada para acceder a un atributo especificado.

```ruby
email_field(:user, :email)
# => <input type="email" id="user_email" name="user[email]" value="#{@user.email}" />
```



### 6.8.12 url\_field

Devuelve una etiqueta de entrada del tipo "url" adaptada para acceder a un atributo especificado.

```ruby
url_field(:user, :url)
# => <input type="url" id="user_url" name="user[url]" value="#{@user.url}" />
```



