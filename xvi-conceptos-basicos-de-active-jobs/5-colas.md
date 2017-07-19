# 5- Colas

La mayoría de los adaptadores admiten varias colas. Con Active Job puede programar el trabajo para que se ejecute en una cola específica:

```ruby
class GuestsCleanupJob < ApplicationJob
  queue_as :low_priority
  #....
end
```

Puede prefijar el nombre de la cola para todos sus trabajos utilizando `config.active_job.queue_name_prefix` en `application.rb`:

```ruby
# config/application.rb
module YourApp
  class Application < Rails::Application
    config.active_job.queue_name_prefix = Rails.env
  end
end
 
# app/jobs/guests_cleanup_job.rb
class GuestsCleanupJob < ApplicationJob
  queue_as :low_priority
  #....
end
 
# Ahora su trabajo se ejecutará en la fila production_low_priority en su
# entorno de producción y en staging_low_priority
# en su entorno de staging
```

El delimitador de prefijo de nombre de cola predeterminado es '`_`'. Esto se puede cambiar configurando `config.active_job.queue_name_delimiter `en `application.rb`:

```ruby
# config/application.rb
module YourApp
  class Application < Rails::Application
    config.active_job.queue_name_prefix = Rails.env
    config.active_job.queue_name_delimiter = '.'
  end
end
 
# app/jobs/guests_cleanup_job.rb
class GuestsCleanupJob < ApplicationJob
  queue_as :low_priority
  #....
end
 
# Ahora su trabajo se ejecutará en la fila production.low_priority en su
# entorno de producción y en staging.low_priority
# en su entorno de staging
```

Si desea más control sobre en qué cola se ejecutará un trabajo, puede pasar una opción `:queue` a `:#set`:

```ruby
MyJob.set(queue: :another_queue).perform_later(record)
```

Para controlar la cola desde el nivel del job, puede pasar un bloque a `#queue_as`. El bloque se ejecutará en el contexto del job \(para poder acceder a `self.arguments`\) y debe devolver el nombre de la cola:

```ruby
class ProcessVideoJob < ApplicationJob
  queue_as do
    video = self.arguments.first
    if video.owner.premium?
      :premium_videojobs
    else
      :videojobs
    end
  end
 
  def perform(video)
    # Do process video
  end
end
 
ProcessVideoJob.perform_later(Video.last)
```

> Asegúrese de que el servidor de cola "escucha" el nombre de su cola. Para algunos backends es necesario especificar las colas para escuchar.



