# 6- Cookies

Su aplicación puede almacenar pequeñas cantidades de datos en el cliente, llamadas `cookies`, que se mantendrán en las solicitudes e incluso en las sesiones. Rails proporciona un fácil acceso a las cookies a través del método `cookies`, que - al igual que la sesión - funciona como un hash:

```ruby
class CommentsController < ApplicationController
  def new
    # Auto-fill the commenter's name if it has been stored in a cookie
    @comment = Comment.new(author: cookies[:commenter_name])
  end
 
  def create
    @comment = Comment.new(params[:comment])
    if @comment.save
      flash[:notice] = "Thanks for your comment!"
      if params[:remember_name]
        # Remember the commenter's name.
        cookies[:commenter_name] = @comment.author
      else
        # Delete cookie for the commenter's name cookie, if any.
        cookies.delete(:commenter_name)
      end
      redirect_to @comment.article
    else
      render action: "new"
    end
  end
end
```

Tenga en cuenta que mientras que para los valores de sesión se establece la key a `nil`, para eliminar un valor de cookie debe utilizar `cookies.delete (:key)`.

Rails también proporciona un cookie jar \(tarro de cookies\) firmado \(signed\) y un tarro de galletas encriptado para almacenar datos confidenciales. El cookie firmado anexa una firma criptográfica en los valores de cookie para proteger su integridad. El tarro de cookies cifrado cifra los valores además de firmarlos, de modo que no puedan ser leídos por el usuario final. Consulte la documentación de la API para obtener más detalles.

Estos tarros de galletas especiales utilizan un serializador para serializar los valores asignados en cadenas y los deserializa en objetos Ruby para la lectura.

Puede especificar qué serializador utilizar:

```ruby
Rails.application.config.action_dispatch.cookies_serializer = :json
```

El serializador predeterminado para las nuevas aplicaciones es `:json`. Para compatibilidad con aplicaciones antiguas con cookies existentes, `:marshal` se utiliza cuando no se especifica la opción serializador.

También puede establecer esta opción en `:hybrid`, en cuyo caso Rails deserializaría de forma transparente las cookies existentes \(`Marshal`-serialized\) en lectura y volver a escribirlas en el formato `JSON`. Esto es útil para migrar aplicaciones existentes a `:json` serializer.

También es posible pasar un serializador personalizado que responde a carga y volcado \(load and dump\):

```ruby
Rails.application.config.action_dispatch.cookies_serializer = MyCustomSerializer
```

Cuando utilice el serializador `:json` o `:hybrid`, debe tener cuidado de que no todos los objetos de Ruby se pueden serializar como `JSON`. Por ejemplo, los objetos Fecha y Hora se serializarán como cadenas y los Hashes tendrán sus claves constreñidas \(stringified\).

```ruby
class CookiesController < ApplicationController
  def set_cookie
    cookies.encrypted[:expiration_date] = Date.tomorrow # => Thu, 20 Mar 2014
    redirect_to action: 'read_cookie'
  end
 
  def read_cookie
    cookies.encrypted[:expiration_date] # => "2014-03-20"
  end
end
```

Es aconsejable que sólo almacene datos sencillos \(cadenas y números\) en las cookies. Si tiene que almacenar objetos complejos, necesitará manejar la conversión manualmente al leer los valores en solicitudes posteriores.

Si utiliza el almacén de sesión de cookies, esto también se aplicaría a `session` y al hash de `flash`.



