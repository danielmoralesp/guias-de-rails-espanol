# 3- Creación de un Job

Esta sección proporcionará una guía paso a paso para crear un job y ponerlo en cola.

### 3.1 Crear el Job

Active Job proporciona un generador de Rails para crear trabajos. Lo siguiente creará un job en `app/jobs` \(con un caso de test adjunto en `test/jobs`\):

```ruby
$ bin/rails generate job guests_cleanup
invoke  test_unit
create    test/jobs/guests_cleanup_job_test.rb
create  app/jobs/guests_cleanup_job.rb
```

También puede crear un trabajo que se ejecutará en una cola específica:

```ruby
$ bin/rails generate job guests_cleanup --queue urgent
```

Si no desea utilizar un generador, puede crear su propio archivo dentro de la app/ trabajos, sólo asegúrese de que hereda de `ApplicationJob`.

Así es como se ve un job:

```ruby
class GuestsCleanupJob < ApplicationJob
  queue_as :default
 
  def perform(*guests)
    # Do something later
  end
end
```

Tenga en cuenta que puede definir `perform` con tantos argumentos como desee.

### 3.2 Encolando el job

Encole un job así:

```ruby
# Encolara un job que se realizará tan pronto como el sistema de colas este libre
GuestsCleanupJob.perform_later guest
```

```ruby
# Encole un job que se ejecutará mañana al mediodía.
GuestsCleanupJob.set(wait_until: Date.tomorrow.noon).perform_later(guest)
```

```ruby
# Encole un job que se realizará en una semana a partir de ahora.
GuestsCleanupJob.set(wait: 1.week).perform_later(guest)
```

```ruby
# `perform_now` y` perform_later` llamarán `perform` bajo la campana para que puedas pasar 
# tantos argumentos como se define en el segundo.
GuestsCleanupJob.perform_later(guest1, guest2, filter: 'some_filter')
```

Eso es todo!







