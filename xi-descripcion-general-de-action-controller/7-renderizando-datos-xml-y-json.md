# 7- Renderizando datos XML y JSON

ActionController facilita enormemente el procesamiento de datos `XML` o `JSON`. Si has generado un controlador usando scaffolding, se vería algo como esto:

```ruby
class UsersController < ApplicationController
  def index
    @users = User.all
    respond_to do |format|
      format.html # index.html.erb
      format.xml  { render xml: @users}
      format.json { render json: @users}
    end
  end
end
```

Puede notar en el código anterior que estamos utilizando `render xml: @users`, no `render xml: @users.to_xml`. Si el objeto no es una cadena, Rails invocará automáticamente `to_xml` por nosotros.

