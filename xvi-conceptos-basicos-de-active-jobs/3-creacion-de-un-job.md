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

Si no desea utilizar un generador, puede crear su propio archivo dentro de la app/ trabajos, sólo asegúrese de que hereda de ApplicationJob.



Así es como se ve un trabajo:

