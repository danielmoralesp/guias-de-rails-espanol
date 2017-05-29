# 11. Bloqueo de registros para actualización

El bloqueo es útil para prevenir condiciones de carrera al actualizar registros en la base de datos y garantizar actualizaciones atómicas.

Active Record ofrece dos mecanismos de bloqueo:

* Bloqueo optimista
* Bloqueo pesimista



### 11.1 Bloqueo optimista

El bloqueo optimista permite a varios usuarios acceder al mismo registro para ediciones y asume un mínimo de conflictos con los datos. Esto lo hace comprobando si otro proceso ha realizado cambios en un registro desde que se abrió. Se produce una excepción `ActiveRecord::StaleObjectError` si se ha producido y la actualización se ignora.

Columna de bloqueo optimista

Para utilizar el bloqueo optimista, la tabla necesita tener una columna llamada `lock_version` del tipo `integer`. Cada vez que se actualiza el registro, el registro activo incrementa la columna `lock_version`. Si se realiza una solicitud de actualización con un valor inferior en el campo `lock_version` que está actualmente en la columna `lock_version` de la base de datos, la solicitud de actualización fallará con `ActiveRecord::StaleObjectError`. Ejemplo:

```ruby
c1 = Client.find(1)
c2 = Client.find(1)
 
c1.first_name = "Michael"
c1.save
 
c2.name = "should fail"
c2.save # Raises an ActiveRecord::StaleObjectError
```

Usted es responsable de resolver el conflicto rescatando la excepción y retrotrayendo, fusionando o aplicando la lógica comercial necesaria para resolver el conflicto.

Este comportamiento se puede desactivar estableciendo `ActiveRecord::Base.lock_optimistically=false.`

Para reemplazar el nombre de la columna `lock_version`, `ActiveRecord::Base` proporciona un atributo de clase denominado `locking_column:`

```ruby
class Client < ApplicationRecord
  self.locking_column = :lock_client_column
end
```



### 11.2 Bloqueo pesimista

El bloqueo pesimista utiliza un mecanismo de bloqueo proporcionado por la base de datos subyacente. Usar `lock` al crear una relación obtiene un bloqueo exclusivo en las filas seleccionadas. Las relaciones que usan `lock` normalmente se envuelven dentro de una transacción para prevenir condiciones de bloqueo.

Por ejemplo:

```ruby
Item.transaction do
  i = Item.lock.first
  i.name = 'Jones'
  i.save!
end
```

La sesión anterior produce el siguiente `SQL` para un backend de `MySQL`:

```ruby
SQL (0.2ms)   BEGIN
Item Load (0.3ms)   SELECT * FROM `items` LIMIT 1 FOR UPDATE
Item Update (0.4ms)   UPDATE `items` SET `updated_at` = '2009-02-07 18:05:56', `name` = 'Jones' WHERE `id` = 1
SQL (0.8ms)   COMMIT
```

También puede pasar el `SQL` sin procesar al método de bloqueo para permitir diferentes tipos de bloqueos. Por ejemplo, `MySQL` tiene una expresión llamada `LOCK IN SHARE MODE` donde puede bloquear un registro pero aún así permitir que otras consultas lo lean. Para especificar esta expresión simplemente entre en la opción `lock`:

```ruby
Item.transaction do
  i = Item.lock("LOCK IN SHARE MODE").find(1)
  i.increment!(:views)
end
```

Si ya tiene una instancia de su modelo, puede iniciar una transacción y adquirir el bloqueo de una vez usando el siguiente código:

```ruby
item = Item.first
item.with_lock do
  # This block is called within a transaction,
  # item is already locked.
  item.increment!(:views)
end
```





