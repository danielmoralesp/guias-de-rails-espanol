# 5. CRUD: Leyendo y Escribiendo Datos

CRUD es el acrónimo para las cuatro verbos que utilizamos para operar con los datos: Create, Read, Update and Delete. Active Record automáticamente crea métodos que permiten a una aplicación leer y manipular los datos guardados dentro de las tablas.

### 5.1 Create

Los objetos Active Record pueden crearse desde un hash, un bloque o configurar sus atributos manualmente antes de la creación. El método `new` retornará un nuevo objeto mientras `create` retornará el objeto y lo guardará en la base de datos.

Por ejemplo, dado un modelo `User` con los atributos `name` y `occupation`, el método llamado `create` creará y guardará un nuevo registro en la base de datos:

```ruby
user = User.create(name: "David", occupation: "Code Artist")
```

Utilizando el método `new`, un objeto puede ser instanciado sin haber sido guardado:

```ruby
user = User.new
user.name = "David"
user.occupation = "Code Artist"
```

Una llamada a `user.save` guardará el registro en la base de datos.

Finalmente, si un bloque es proveído, ambos `create` y `new` producirá el nuevo objeto inicializado dentro de un bloque:

```ruby
user = User.new do |u|
  u.name = "David"
  u.occupation = "Code Artist"
end
```

### 5.2 Read

Active Record provee una rica API para acceder a los datos dentro de una base de datos. Debajo hay algunos ejemplos de diferentes métodos provistos por Active Record.

```ruby
# devuelve una colección de usuarios
users = User.all
# devuelve el primer usuario
user = User.first
# devuelve el primer usuario llamado David
david = User.find_by(name: 'David')
# encontrar todos los usuarios llamados David que tienen de ocupación Code Artists y ordenado por created_at en sentido cronológicamente inverso
users = User.where(name: 'David', occupation: 'Code Artist').order('created_at DESC')
```

Puedes aprender más acerca de consultar un modelo Active Record en la guía Interface de Consultas Active Record.

### 5.3 Update

Una vez que un objeto Active Record ha sido recuperado, sus atributos pueden ser modificados y volver a ser guardados en la base de datos.

```ruby
user = User.find_by(name: 'David')
user.name = 'Dave'
user.save
```

Un camino corto para este uso es mapear un hash con los nombres de los atributos que se desean modificar y su valor, como por ejemplo:

```ruby
user = User.find_by(name: 'David')
user.update(name: 'Dave')
```

Esta es la forma más poderosa de actualizar varios atributos a la vez. Si, por otro lado, quieres actualizar varios registros a la vez, encontrarás muy útil el método de clase `update_all`:

`User.update_all "max_login_attempts = 3, must_change_password = 'true'"`

### 5.4 Delete

Asimismo, una vez que se recupera el objeto Active Record también puede ser destruído, lo cual lo borrará de la base de datos.

```ruby
user = User.find_by(name: 'David')
user.destroy
```



