# 6.10 FormTagHelper

Proporciona una serie de métodos para crear etiquetas de formulario que no dependen de un objeto Active Record asignado a la plantilla como lo hace FormHelper. En su lugar, proporciona los nombres y los valores manualmente.



### 6.10.1 check\_box\_tag

Crea una etiqueta de entrada de formulario de casilla de verificación.

```ruby
check_box_tag 'accept'
# => <input id="accept" name="accept" type="checkbox" value="1" />
```



### 6.10.2 field\_set\_tag

Crea un conjunto de campos para agrupar elementos de formulario HTML.

```ruby
<%= field_set_tag do %>
  <p><%= text_field_tag 'name' %></p>
<% end %>
# => <fieldset><p><input id="name" name="name" type="text" /></p></fieldset>
```



### 6.10.3 file\_field\_tag

Crea un campo de carga de archivos.

```ruby
<%= form_tag({ action: "post" }, multipart: true) do %>
  <label for="file">File to Upload</label> <%= file_field_tag "file" %>
  <%= submit_tag %>
<% end %>
```

Ejemplo de salida:

```ruby
file_field_tag 'attachment'
# => <input id="attachment" name="attachment" type="file" />
```



### 6.10.4 form\_tag

Inicia una etiqueta de formulario que señala el `action` a una URL configurada con `url_for_options` como `ActionController::Base#url_for`.

```ruby
<%= form_tag '/articles' do %>
  <div><%= submit_tag 'Save' %></div>
<% end %>
# => <form action="/articles" method="post"><div><input type="submit" name="submit" value="Save" /></div></form>
```



### 6.10.5 hidden\_field\_tag

Crea un campo de entrada de formulario oculto que se utiliza para transmitir datos que se perderían debido a la apatridia de HTTP o datos que deberían estar ocultos al usuario.

```ruby
hidden_field_tag 'token', 'VUBJKB23UIVI1UU1VOBVI@'
# => <input id="token" name="token" type="hidden" value="VUBJKB23UIVI1UU1VOBVI@" />
```



### 6.10.6 image\_submit\_tag

Muestra una imagen que, cuando se hace clic, enviará el formulario.

```ruby
image_submit_tag("login.png")
# => <input src="/images/login.png" type="image" />
```



### 6.10.7 label\_tag

Crea un campo de etiqueta.

```ruby
label_tag 'name'
# => <label for="name">Name</label>
```



### 6.10.8 password\_field\_tag

Crea un campo de contraseña, un campo de texto enmascarado que oculta la entrada de los usuarios detrás de un carácter de máscara.

```ruby
password_field_tag 'pass'
# => <input id="pass" name="pass" type="password" />
```



### 6.10.9 radio\_button\_tag

Crea un radio button; Utilice grupos de radio buttons con el mismo nombre para permitir a los usuarios seleccionar de un grupo de opciones.

```ruby
radio_button_tag 'gender', 'male'
# => <input id="gender_male" name="gender" type="radio" value="male" />
```



### 6.10.10 select\_tag

Crea un cuadro de selección desplegable.

```ruby
select_tag "people", "<option>David</option>"
# => <select id="people" name="people"><option>David</option></select>
```



### 6.10.11 submit\_tag

Crea un botón de envío con el texto proporcionado como el título.

```ruby
submit_tag "Publish this article"
# => <input name="commit" type="submit" value="Publish this article" />
```



### 6.10.12 text\_area\_tag

Crea un área de entrada de texto; Utilice una zona de texto para entradas de texto más largas, como publicaciones o descripciones de blogs.

```ruby
text_area_tag 'article'
# => <textarea id="article" name="article"></textarea>
```



### 6.10.13 text\_field\_tag

Crea un campo de texto estándar; Utilice estos campos de texto para introducir fragmentos más pequeños de texto como un nombre de usuario o una consulta de búsqueda.

```ruby
text_field_tag 'name'
# => <input id="name" name="name" type="text" />
```



### 6.10.14 email\_field\_tag

Crea un campo de entrada estándar del tipo email.

```ruby
email_field_tag 'email'
# => <input id="email" name="email" type="email" />
```



### 6.10.15 url\_field\_tag

Crea un campo de entrada estándar de tipo url.

```ruby
url_field_tag 'url'
# => <input id="url" name="url" type="url" />
```



### 6.10.16 date\_field\_tag

Crea un campo de entrada estándar de tipo fecha.

```ruby
date_field_tag "dob"
# => <input id="dob" name="dob" type="date" />
```



