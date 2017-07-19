# 10- Excepciones

Active Job proporciona una forma de detectar excepciones generadas durante la ejecución del job:

```ruby
class GuestsCleanupJob < ApplicationJob
  queue_as :default
 
  rescue_from(ActiveRecord::RecordNotFound) do |exception|
   # Do something with the exception
  end
 
  def perform
    # Do something later
  end
end
```

### 10.1 Deserialización

GlobalID permite serializar los objetos Active Record completos pasados a `#perform`.

Si se elimina un registro pasado después de que se encola el job pero antes de que el método `#perform` es llamado , Active Job generará una excepción `ActiveJob::DeserializationError`.



