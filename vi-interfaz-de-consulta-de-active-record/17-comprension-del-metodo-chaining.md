# 17. Comprensión del método Chaining

El patrón Active Record implementa el método `Chaining`, que nos permite usar múltiples métodos de Active Record juntos de una manera sencilla y directa.

Puede encadenar métodos en una instrucción cuando el método anterior devuelve un ActiveRecord::Relation, como `all`, `where` y `join`. Los métodos que devuelven un único objeto \(consulte como recuperar datos de un solo objeto\) deben estar al final de la instrucción.

Hay algunos ejemplos a continuación. Esta guía no cubrirá todas las posibilidades, sólo algunas como ejemplos. Cuando se llama a un método de Active Record, la consulta no se genera inmediatamente y se envía a la base de datos, esto sólo sucede cuando los datos son realmente necesitados. Así, cada ejemplo a continuación genera una sola consulta.



### 17.1 Recuperación de datos filtrados de  multiples tablas

```ruby
Person
  .select('people.id, people.name, comments.text')
  .joins(:comments)
  .where('comments.created_at > ?', 1.week.ago)
```

El resultado debe ser algo como esto:

```ruby
SELECT people.id, people.name, comments.text
FROM people
INNER JOIN comments
  ON comments.person_id = people.id
WHERE comments.created_at > '2015-01-01'
```



### 17.2 Recuperación de datos específicos de varias tablas

```ruby
Person
  .select('people.id, people.name, companies.name')
  .joins(:company)
  .find_by('people.name' => 'John') # Esta debe ser la última
```

Lo anterior debería generar:

```ruby
SELECT people.id, people.name, companies.name
FROM people
INNER JOIN companies
  ON companies.person_id = people.id
WHERE people.name = 'John'
LIMIT 1
```

> Tenga en cuenta que si una consulta coincide con varios registros, `find_by` buscará sólo el primero e ignorará los demás \(consulte la instrucción `LIMIT 1` anterior\).



