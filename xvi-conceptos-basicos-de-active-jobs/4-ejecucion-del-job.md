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

### 4.3 Inicio del backend

Dado que los jobs se ejecutan en paralelo con la aplicación de Rails, la mayoría de las bibliotecas de colas requieren que se inicie un servicio de colas específico de la biblioteca \(además de iniciar la aplicación Rails\) para que el job funcione. Consulte la documentación de la biblioteca para obtener instrucciones sobre cómo iniciar el backend de la cola.

Aquí hay una lista no exhaustiva de documentación:

* [Sidekiq](https://github.com/mperham/sidekiq/wiki/Active-Job)
* [Resque](https://github.com/resque/resque/wiki/ActiveJob)
* [Sucker Punch](https://github.com/brandonhilkert/sucker_punch#active-job)
* [Queue Classic](https://github.com/QueueClassic/queue_classic#active-job)



