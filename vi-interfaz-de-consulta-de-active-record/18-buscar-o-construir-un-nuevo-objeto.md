# 18. Buscar o construir un nuevo objeto

Es común que necesite encontrar un registro o crearlo si no existe. Puede hacerlo con los métodos `find_or_create_by` o con `find_or_create_by!` .



### 18.1 find\_or\_create\_by

El método `find_or_create_by` comprueba si existe un registro con los atributos especificados. Si no lo hace, entonces se llama a `create`. Veamos un ejemplo.

Suponga que desea buscar un cliente llamado 'Andy', y si no hay ninguno, cree uno. Puede hacerlo ejecutando:

```ruby
Client.find_or_create_by(first_name: 'Andy')
# => #<Client id: 1, first_name: "Andy", orders_count: 0, locked: true, created_at: "2011-08-30 06:09:27", updated_at: "2011-08-30 06:09:27">
```

El SQL generado por este método se ve así:

```ruby
SELECT * FROM clients WHERE (clients.first_name = 'Andy') LIMIT 1
BEGIN
INSERT INTO clients (created_at, first_name, locked, orders_count, updated_at) VALUES ('2011-08-30 05:22:57', 'Andy', 1, NULL, '2011-08-30 05:22:57')
COMMIT
```

`find_or_create_by` devuelve el registro que ya existe o el nuevo registro. En nuestro caso, no tenemos un cliente llamado Andy por lo que el registro se crea y lo devuelve.

Es posible que el nuevo registro no se guarde en la base de datos; Que depende de si las validaciones pasaron o no \(al igual que crear\).

Supongamos que queremos poner el atributo '`locked`' en `false` si estamos creando un nuevo registro, pero no queremos incluirlo en la consulta. Así que queremos encontrar el cliente llamado "Andy", o si ese cliente no existe, crear un cliente llamado "Andy" que no está bloqueado.

Podemos lograr esto de dos maneras. La primera es usar `create_with`:

```ruby
Client.create_with(locked: false).find_or_create_by(first_name: 'Andy')
```

La segunda forma es usar un bloque:

```ruby
Client.find_or_create_by(first_name: 'Andy') do |c|
  c.locked = false
end
```

El bloque sólo se ejecutará si se está creando el cliente. La segunda vez que ejecutamos este código, el bloque será ignorado.



### 18.2 find\_or\_create\_by!

También puede utilizar `find_or_create_by!` Para generar una excepción si el nuevo registro no es válido. Las validaciones no están cubiertas en esta guía, pero supongamos por un momento que las agregamos temporalmente

```ruby
validates :orders_count, presence: true
```

A su modelo de cliente. Si intenta crear un nuevo cliente sin pasar un `orders_count`, el registro no será válido y se generará una excepción:

```ruby
Client.find_or_create_by!(first_name: 'Andy')
# => ActiveRecord::RecordInvalid: Validation failed: Orders count can't be blank
```



### 18.3 find\_or\_initialize\_by

El método `find_or_initialize_by` funcionará igual que `find_or_create_by` pero llamará `new` en lugar de `create`. Esto significa que se creará una nueva instancia del modelo en la memoria pero no se guardará en la base de datos. Continuando con el ejemplo `find_or_create_by`, ahora queremos que el nombre del cliente sea 'Nick':

```ruby
nick = Client.find_or_initialize_by(first_name: 'Nick')
# => #<Client id: nil, first_name: "Nick", orders_count: 0, locked: true, created_at: "2011-08-30 06:09:27", updated_at: "2011-08-30 06:09:27">
 
nick.persisted?
# => false
 
nick.new_record?
# => true
```







