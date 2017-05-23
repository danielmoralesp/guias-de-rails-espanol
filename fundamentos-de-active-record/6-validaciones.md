# 6. Validaciones

Active Record te permite validar el estado del modelo antes que se escriba en la base de datos. Hay varios métodos que puedes utilizar para comprobar tu modelo y validar que un atributo no está vacío, es único y no está aún en la base de datos, que siga un formato específico, y muchos más. La validación es una tarea muy importante a considerar cuando se guardan datos en una base de datos, entonces los métodos `save` y `update` toman esto en cuenta cuando se ejecutan: ellos retornan `false` cuando una validación falla y no mantuvieron ninguna operación en la base de datos. Todos estos tienen una contrapartida bang \(esta es , `save!` y `update!`\), las cuales son estrictas y arrojan una excepciónn `ActiveRecord::RecordInvalid` si la validación falla. Un rápido ejemplo para ilustralo:

```ruby
class User < ActiveRecord::Base
  validates :name, presence: true
end
 
user = User.new
user.save  # => false
user.save! # => ActiveRecord::RecordInvalid: Validation failed: Name can't be blank
```

Puedes aprender más acerca de validaciones en la Guía de Validaciones Active Record.

