# 13- Recursos Adicionales de Pruebas

### 13.1 Prueba del código dependiente del tiempo

Rails proporciona métodos auxiliares integrados que le permiten afirmar que su código sensible al tiempo funciona de la manera esperada.

Aquí hay un ejemplo usando el ayudante  `travel_to`:

```ruby
# Lets say that a user is eligible for gifting a month after they register.
user = User.create(name: 'Gaurish', activation_date: Date.new(2004, 10, 24))
assert_not user.applicable_for_gifting?
travel_to Date.new(2004, 11, 24) do
  assert_equal Date.new(2004, 10, 24), user.activation_date # inside the `travel_to` block `Date.current` is mocked
  assert user.applicable_for_gifting?
end
assert_equal Date.new(2004, 10, 24), user.activation_date # The change was visible only inside the `travel_to` block.
```

Consulte las [documentación de la API](http://api.rubyonrails.org/v5.1.2/classes/ActiveSupport/Testing/TimeHelpers.html) de  `ActiveSupport::Testing::TimeHelpers` para obtener información detallada sobre los ayudantes de tiempo disponibles.

