# 5- Problemas con el servidor

Ajax no es sólo `client-side`, usted también necesita hacer un cierto trabajo en el lado del servidor para apoyarlo. A menudo, la gente hace sus solicitudes Ajax para devolver JSON en lugar de HTML. Vamos a discutir lo que se necesita para que eso suceda.

### 5.1 Un ejemplo simple

Imagine que tiene una serie de usuarios que desea mostrar y proporciona un formulario en esa misma página para crear un nuevo usuario. La acción `index` de su controlador se parece a esto:

```ruby
class UsersController < ApplicationController
  def index
    @users = User.all
    @user = User.new
  end
  # ...
```

La vista `index` \(`app/views/users/index.html.erb`\) contiene:

```html
<b>Users</b>
 
<ul id="users">
<%= render @users %>
</ul>
 
<br>
 
<%= form_with(model: @user) do |f| %>
  <%= f.label :name %><br>
  <%= f.text_field :name %>
  <%= f.submit %>
<% end %>
```

El partial   `app/views/users/_user.html.erb`  contiene lo siguiente:

```ruby
<li><%= user.name %></li>
```

La parte superior de la página `index` muestra los usuarios. La parte inferior proporciona un formulario para crear un nuevo usuario.

El formulario inferior llamará a la acción `create` del `UsersController`. Como la opción `remote` del formulario está establecida en `true`, la solicitud se publicará en el `UsersController` como una solicitud de Ajax, buscando JavaScript. Con el fin de atender a esta solicitud, la acción `create` de su controlador se vería así:

```ruby
# app/controllers/users_controller.rb
# ......
def create
  @user = User.new(params[:user])
 
  respond_to do |format|
    if @user.save
      format.html { redirect_to @user, notice: 'User was successfully created.' }
      format.js
      format.json { render json: @user, status: :created, location: @user }
    else
      format.html { render action: "new" }
      format.json { render json: @user.errors, status: :unprocessable_entity }
    end
  end
end
```

Observe el `format.js` en el bloque `respond_to`; Que permite al controlador responder a su solicitud de Ajax. A continuación, tiene un archivo de vista correspondiente que genera el código JavaScript real que se enviará y ejecutará en el lado del cliente en `app/views/users/create.js.erb` .

```js
$("<%= escape_javascript(render @user) %>").appendTo("#users");
```



