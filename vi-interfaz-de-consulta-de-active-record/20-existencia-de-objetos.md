# 20. Existencia de Objetos

Si simplemente desea comprobar la existencia del objeto hay un método llamado `exist?`. Este método consultará la base de datos utilizando la misma consulta que `find`, pero en lugar de devolver un objeto o una colección de objetos, devolverá `true` o `false`.

```ruby
Client.exists?(1)
```

El método `exist?` también toma varios valores, pero al capturarlo devolverá `true` si existe alguno de esos registros.

```ruby
Client.exists?(id: [1,2,3])
# or
Client.exists?(name: ['John', 'Sergei'])
```

Incluso es posible utilizar `exist?` sin ningún argumento sobre un modelo o una relación.

```ruby
Client.where(first_name: 'Ryan').exists?
```

Lo anterior devuelve `true` si hay al menos un cliente con el `first_name` 'Ryan' y `false` en caso contrario.

```ruby
Client.exists?
```

Lo anterior devuelve `false` si la tabla de clientes está vacía y `true` en caso contrario.

También puede utilizar `any?` y `many?` Para verificar la existencia de un modelo o relación.

```ruby
# via a model
Article.any?
Article.many?
 
# via a named scope
Article.recent.any?
Article.recent.many?
 
# via a relation
Article.where(published: true).any?
Article.where(published: true).many?
 
# via an association
Article.first.categories.any?
Article.first.categories.many?
```



