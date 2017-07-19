# 9- GlobalID

Active Job admite GlobalID para los parámetros. Esto hace posible pasar los objetos vivos de Active Record a su job en vez de los pares de class/id, que entonces usted tendría que deserializar manualmente. Antes, los jobs se veían así:

```ruby
class TrashableCleanupJob < ApplicationJob
  def perform(trashable_class, trashable_id, depth)
    trashable = trashable_class.constantize.find(trashable_id)
    trashable.cleanup(depth)
  end
end
```

Ahora usted puede simplemente hacer:

```ruby
class TrashableCleanupJob < ApplicationJob
  def perform(trashable, depth)
    trashable.cleanup(depth)
  end
end
```

Esto funciona con cualquier clase que se mezcla en `GlobalID::Identification`, que por defecto se ha mezclado en las clases Active Record.

