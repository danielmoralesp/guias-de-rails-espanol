# 10. Callbacks de Transacciones

Hay dos callbacks adicionales que serán disparados para completar una transacción de base de datos: `after_commit` y `after_rollback`. Estos callbacks son muy parecidos al callback `after_save` excepto que estos no se ejecutan hasta después de los cambios en la base de datos hayan sido commiteados o revertidos. Son muy útiles cuando tus modelos active record necesitan interactuar con sistemas externos los cuales no son parte de una transacción de la base de datos. 

Considera, por ejemplo, los ejemplos previos donde el modelo `PictureFile` necesita borrar un fichero después de que el registro correspondiente es destruído. Si por algo se lanzase una excepción después del callback `after_destroy` y la transacción se revirtiera, el fichero se habrá borrado y dejará el modelo en un estado inconsistente. Por ejemplo, suponiendo que `picture_file_2` en el código de abajo no es válido y el método `save!` arroja un error.

```ruby
PictureFile.transaction do
  picture_file_1.destroy
  picture_file_2.save!
end
```

Utilizando el callback `after_commit` podemos solucionar este caso.

```ruby
class PictureFile < ActiveRecord::Base
  after_commit :delete_picture_file_from_disk, on: [:destroy]
 
  def delete_picture_file_from_disk
    if File.exist?(filepath)
      File.delete(filepath)
    end
  end
end
```

La opción `:on` especifica cuando un callback será disparado. Si tu no escribes la opción `:on` el callback será disparado en todas las acciones.

Los callbacks `after_commit` y `after_rollback` nos garantizan que serán llamados por todos los modelos creados, actualizados, o destuídos, dentro de un bloque de transacción. Si alguna excepción es lanzada dentro de estos callbacks, serán ignorados y entonces no interfieren con otros callbacks. Así como también, si tu código callback puede lanzar una excepción, necesitarás rescatarla y manejarla apropiadamente dentro del callback.

