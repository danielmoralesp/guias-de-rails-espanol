# 4- Ejecución del Job

Para encolar y ejecutar jobs en producción, debe configurar un backend de colas, es decir, debe decidirse por una biblioteca de colas de terceros que Rails debería usar. Rails solo proporciona un sistema de colas en proceso, que sólo mantiene los trabajos en la RAM. Si el proceso se bloquea o se restablece la máquina, todos los trabajos pendientes se pierden con el backend asíncrono predeterminado. Esto puede estar bien para aplicaciones más pequeñas o trabajos no críticos, pero la mayoría de las aplicaciones en producción tendrán que escoger un backend persistente.

### 4.1 Backends

Active Job tiene adaptadores integrados para múltiples backups de cola \(Sidekiq, Resque, Delayed Job y otros\). Para obtener una lista actualizada de los adaptadores, consulte la documentación de la API para [`ActiveJob::QueueAdapters`](http://api.rubyonrails.org/v5.1.2/classes/ActiveJob/QueueAdapters.html).

### 4.2 Configuración del backend

Puede configurar fácilmente el backend de colas de esta manera:

```ruby
# config/application.rb
module YourApp
  class Application < Rails::Application
    # Asegúrate de tener la gema del adaptador en tu Gemfile
    # y siga la instalación específica del adaptador
    # y las instrucciones de implementación.
    config.active_job.queue_adapter = :sidekiq
  end
end
```

También puede configurar su back-end por cada Job.

```ruby
class GuestsCleanupJob < ApplicationJob
  self.queue_adapter = :resque
  #....
end
 
# Ahora su job utilizará `resque` ya que es el adaptador de cola de backend que sobre-escribe lo que
# se ha configurado en `config.active_job.queue_adapter`.
```



