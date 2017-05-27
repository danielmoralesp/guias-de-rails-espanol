# 2. Condiciones

El método `where` permite especificar condiciones para limitar los registros devueltos, que representan la parte `WHERE` de la sentencia `SQL`. Las condiciones se pueden especificar como una cadena, una matriz o un hash.



### 2.1 Condiciones con cadenas puras

Si desea agregar condiciones a su búsqueda, sólo debe especificarlas allí, al igual que `Client.where("orders_count = '2'")`. Esto encontrará todos los clientes donde el valor del campo `orders_count` es 2.

> Construir sus propias condiciones como cadenas puras puede dejarlo vulnerable a las explotaciones de SQL Injections. Por ejemplo, `Client.where("first_name LIKE '%#{params[:first_name]}%'")` no es seguro. Consulte la sección siguiente para conocer la forma preferida de manejar las condiciones usando una matriz.



### 2.2 Condiciones con Arrays

¿Y si ese número pudiera variar, digamos como un argumento de alguna parte? La búsqueda tomaría entonces la forma:

```ruby
Client.where("orders_count = ?", params[:orders])
```

Active Record tomará el primer argumento como la cadena de condiciones y cualquier argumento adicional lo reemplazará con los signos de interrogación \(?\)

Si desea especificar varias condiciones:

```ruby
Client.where("orders_count = ? AND locked = ?", params[:orders], false)
```

En este ejemplo, el primer signo de interrogación se reemplazará con el valor de `params[:orders]` y el segundo se sustituirá por la representación `SQL` de `false`, que depende del adapter.

Este código es altamente preferible:

```ruby
Client.where("orders_count = ?", params[:orders])
```

A éste código:

```ruby
Client.where("orders_count = #{params[:orders]}")
```

Debido a la seguridad del argumento. Poner la variable directamente en la cadena de condiciones pasará la variable a la base de datos tal como está. Esto significa que será una variable sin escapar directamente de un usuario que puede tener intención maliciosa. Si lo hace, pone toda su base de datos en riesgo, porque una vez que un usuario se entera de que pueden explotar su base de datos, puede hacer casi cualquier cosa con la misma. Nunca pongas tus argumentos directamente dentro de la cadena de condiciones.

#### 2.2.1 Condiciones "Placeholder"

Similar al estilo de reemplazo \(?\) de los parámetros, también puede especificar las claves en su cadena de condiciones junto con las claves/valores correspondientes al hash:

```ruby
Client.where("created_at >= :start_date AND created_at <= :end_date",
  {start_date: params[:start_date], end_date: params[:end_date]})
```

Esto hace más clara la legibilidad si usted tiene un gran número de condiciones variables.



### 2.3 Condiciones con Hashes

Active Record también le permite pasar condiciones con hash, las cuales pueden aumentar la legibilidad de su sintaxis de condiciones. Con las condiciones hash, usted pasa en un hash las llaves de los campos que usted quiere buscar y los valores de cómo usted desea buscarlos:

> Sólo la igualdad, los rangos y la comprobación de subconjuntos son posibles con las condiciones de Hash.

#### 2.3.1 Condiciones de Igualdad

```ruby
Client.where(locked: true)
```

Esto generará un `SQL` como este:

```ruby
SELECT * FROM clients WHERE (clients.locked = 1)
```

El nombre del campo también puede ser una cadena:

```ruby
Client.where('locked' => true)
```

En el caso de una relación `belongs_to`, se puede utilizar una clave de asociación para especificar el modelo si se utiliza un objeto Active Record como valor. Este método funciona con relaciones polimórficas también.

```ruby
Article.where(author: author)
Author.joins(:articles).where(articles: { author: author })
```











