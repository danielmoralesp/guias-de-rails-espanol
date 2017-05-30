# 15.  Buscadores dinámicos

Para cada campo \(también conocido como un atributo\) que define en su tabla, Active Record proporciona un método `finder`. Si tiene un campo llamado `first_name` en su modelo de cliente, por ejemplo, obtiene `find_by_first_name` gratis desde Active Record. Si tiene un campo bloqueado en el modelo de cliente, también encontrará el método `find_by_locked`.

Puede especificar un punto de exclamación \(!\) al final de los buscadores dinámicos para que generen un error `ActiveRecord::RecordNotFound` si no devuelven registros, como `Client.find_by_name!("Ryan")`

Si desea encontrar ambos por nombre y bloqueo, puede encadenar estos buscadores simplemente escribiendo "`and`" entre los campos. Por ejemplo, `Client.find_by_first_name_and_locked("Ryan", true).`



