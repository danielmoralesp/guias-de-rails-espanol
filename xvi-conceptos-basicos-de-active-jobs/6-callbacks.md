# 6- Callbacks

Trabajo activo proporciona hooks durante el ciclo de vida de un job. Las devoluciones de llamada le permiten activar la lógica durante el ciclo de vida de un trabajo.

### 6.1 Devoluciones de llamada disponibles

* `before_enqueue`
* `around_enqueue`
* `after_enqueue`
* `before_perform`
* `around_perform`
* `after_perform`

### 6.2 Uso

```ruby
class GuestsCleanupJob < ApplicationJob
  queue_as :default
 
  before_enqueue do |job|
    # Do something with the job instance
  end
 
  around_perform do |job, block|
    # Do something before perform
    block.call
    # Do something after perform
  end
 
  def perform
    # Do something later
  end
end
```



