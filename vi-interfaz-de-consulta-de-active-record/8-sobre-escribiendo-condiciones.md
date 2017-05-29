# 8- Sobre-escribiendo condiciones



### 8.1 Unscope

Puede especificar ciertas condiciones que se eliminarán mediante el método unscope. Por ejemplo:

```ruby
Article.where('id > 10').limit(20).order('id asc').unscope(:order)
```

El `SQL` que se ejecutaría:

```ruby
SELECT * FROM articles WHERE id > 10 LIMIT 20
 
# Consulta original sin `unscope`
SELECT * FROM articles WHERE id > 10 ORDER BY id asc LIMIT 20
```

También puede hacer `unscope` sobre cláusulas `where` específicas. Por ejemplo:

```ruby
Article.where(id: 10, trashed: false).unscope(where: :id)
# SELECT "articles".* FROM "articles" WHERE trashed = 0
```

Una relación que ha utilizado el `unscope` afectará cualquier relación en la que se haga `merge`:

```ruby
Article.order('id asc').merge(Article.unscope(:order))
# SELECT "articles".* FROM "articles"
```



### 8.2 only

También puede anular las condiciones utilizando el método `only`. Por ejemplo:

```ruby
Article.where('id > 10').limit(20).order('id desc').only(:order, :where)
```

El `SQL` que se ejecutaría es el siguiente:

```ruby
SELECT * FROM articles WHERE id > 10 ORDER BY id DESC
 
# Consulta original sin 'only`
SELECT "articles".* FROM "articles" WHERE (id > 10) ORDER BY id desc LIMIT 20
```



### 8.3 reorder

El método `reorder` reemplaza el orden del `scope` predeterminado. Por ejemplo:

```ruby
class Article < ApplicationRecord
  has_many :comments, -> { order('posted_at DESC') }
end
 
Article.find(10).comments.reorder('name')
```

El `SQL` que se ejecutaría:

```ruby
SELECT * FROM articles WHERE id = 10
SELECT * FROM comments WHERE article_id = 10 ORDER BY name
```

En el caso en que no se use la cláusula `reorder`, el `SQL` ejecutado sería:

```ruby
SELECT * FROM articles WHERE id = 10
SELECT * FROM comments WHERE article_id = 10 ORDER BY posted_at DESC
```



### 8.4 reverse\_order

El método `reverse_order` invierte la cláusula de ordenación si se especifica.

```ruby
Client.where("orders_count > 10").order(:name).reverse_order
```

El `SQL` que se ejecutaría:

```ruby
SELECT * FROM clients WHERE orders_count > 10 ORDER BY name DESC
```

Si no se especifica ninguna cláusula de orden en la consulta, la `reverse_order` ordena la clave principal en orden inverso.

```ruby
Client.where("orders_count > 10").reverse_order
```

El `SQL` que se ejecutaría:

```ruby
SELECT * FROM clients WHERE orders_count > 10 ORDER BY clients.id DESC
```

Este método no acepta argumentos.



### 8.5 rewhere

El método `rewhere` reemplaza una condición existente, denominada `where`. Por ejemplo:

```ruby
Article.where(trashed: true).rewhere(trashed: false)
```

El `SQL` que se ejecutaría:

```ruby
SELECT * FROM articles WHERE `trashed` = 0
```

En caso de que no se utilice la cláusula `rewhere`,

```ruby
Article.where(trashed: true).where(trashed: false)
```

El `SQL` ejecutado sería:

```ruby
SELECT * FROM articles WHERE `trashed` = 1 AND `trashed` = 0
```













