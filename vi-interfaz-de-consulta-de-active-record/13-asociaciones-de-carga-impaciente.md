# 13. Asociaciones de carga impaciente

La carga ansiosa es el mecanismo para cargar los registros asociados de los objetos devueltos por `Model.find` utilizando el menor número posible de consultas.

Problema de consultas N + 1 

Considere el siguiente código, que encuentra 10 clientes e imprime sus códigos postales:

```ruby
clients = Client.limit(10)

clients.each do |client|
  puts client.address.postcode
end
```

Este código se ve bien a primera vista. Pero el problema está dentro del número total de consultas ejecutadas. El código anterior ejecuta 1 consulta \(para encontrar 10 clientes\) + 10 consultas adicionales \(uno por cada cliente para cargar la dirección\) = 11 consultas en total.

**Solución al problema de consultas N + 1**

Active Record le permite especificar de antemano todas las asociaciones que se van a cargar. Esto es posible especificando el método `includes` de la llamada `Model.find.` Con `includes`, Active Record garantiza que todas las asociaciones especificadas se cargan utilizando el número mínimo posible de consultas.

Revisando el caso anterior, podríamos reescribir `Client.limit(10)` para cargar ansiosamente las direcciones:

```ruby
clients = Client.includes(:address).limit(10)
 
clients.each do |client|
  puts client.address.postcode
end
```

El código anterior ejecutará sólo 2 consultas, en lugar de 11 consultas en el caso anterior:

```ruby
SELECT * FROM clients LIMIT 10
SELECT addresses.* FROM addresses
  WHERE (addresses.client_id IN (1,2,3,4,5,6,7,8,9,10))
```



### 13.1 Carga de múltiples asociaciones

Active Record le permite cargar ansiosamente cualquier número de asociaciones con una sola llamada `Model.find` utilizando un array, hash o un hash anidado de array/hash con el método `includes`.

#### 13.1.1 Arreglo de Asociaciones Múltiples

```ruby
Article.includes(:category, :comments)
```

Esto carga todos los artículos y la categoría asociada y comentarios para cada artículo.

#### 13.1.2 Asociaciones anidadas con Hash

```ruby
Category.includes(articles: [{ comments: :guest }, :tags]).find(1)
```

Esto encontrará la categoría con id 1 y carga ansiosamente todos los artículos asociados, las etiquetas y comentarios de los artículos asociados y la asociación de invitados de cada comentario.



### 13.2 Especificación de condiciones en las asociaciones cargadas ansiosamente

Aunque Active Record le permite especificar condiciones en las asociaciones cargadas ansiosamente como `joins`, la forma recomendada es usar `joins` en su lugar.

Sin embargo, si tiene que hacer esto, puede usarlo como lo haría normalmente.

```ruby
Article.includes(:comments).where(comments: { visible: true })
```

Esto generaría una consulta que contiene un `LEFT OUTER JOIN`, mientras que el método `join` generaría uno utilizando la función `INNER JOIN`.

```ruby
SELECT "articles"."id" AS t0_r0, ... "comments"."updated_at" AS t1_r5 FROM "articles" LEFT OUTER JOIN "comments" ON "comments"."article_id" = "articles"."id" WHERE (comments.visible = 1)
```

Si no había ninguna condición `where`, esto generaría el conjunto normal de dos consultas.

> El uso de `where` sólo funcionará cuando se pasa un Hash. Para los fragmentos SQL, debe utilizar referencias para forzar `joined tables`:



```ruby
Article.includes(:comments).where("comments.visible = true").references(:comments)
```

Si, en este caso se incluye una consulta, y no hubo comentarios para ningún artículo, todos los artículos aún se cargarían. Mediante el uso de `joins` \(un INNER JOIN\), las condiciones de unión deben coincidir, de lo contrario no se devolverán registros.

> Si una asociación está cargada con impaciencia como parte de una `join table`, los campos de una cláusula de selección personalizada no aparecerán en los modelos cargados. Esto es porque es ambiguo si deben aparecer en el registro principal, o en el child.







