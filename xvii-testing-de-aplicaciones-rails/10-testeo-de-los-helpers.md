# 10- Testeo de los Helpers

Un helper es simplemente un módulo donde puedes definir los métodos que estarán disponibles en tus vistas.

Para probar los helpers, todo lo que necesitas hacer es comprobar que la salida del método del helper coincide con lo que esperas. Las pruebas relacionadas con los helpers se encuentran en el directorio `test/helpers`.

Dado que tenemos el siguiente helper:

```ruby
module UserHelper
  def link_to_user(user)
    link_to "#{user.first_name} #{user.last_name}", user
  end
end
```

Podemos probar la salida de este método con algo como esto:

```ruby
class UserHelperTest < ActionView::TestCase
  test "should return the user's full name" do
    user = users(:david)
 
    assert_dom_equal %{<a href="/user/#{user.id}">David Heinemeier Hansson</a>}, link_to_user(user)
  end
end
```

Además, dado que la clase de testing se extiende desde `ActionView::TestCase`, tiene acceso a los métodos auxiliares de Rails como `link_to` o `pluralize`.



