# 5- Inspección y pruebas de las rutas

Rails ofrece facilidades para inspeccionar y probar sus rutas.

### 5.1 Listado de rutas existentes

Para obtener una lista completa de las rutas disponibles en su aplicación, visite `http://localhost:3000/rails/info/routes` en su navegador mientras su servidor se está ejecutando en el entorno de desarrollo. También puede ejecutar el comando `rails routes` en su terminal para producir la misma salida.

Ambos métodos enumerarán todas sus rutas, en el mismo orden en que aparecen en `config/routes.rb`. Para cada ruta, verá:

* El nombre de la ruta \(si existe\)
* El verbo `HTTP` utilizado \(si la ruta no responde a todos los verbos\)
* El patrón de URL que coincide
* Los parámetros de enrutamiento para la ruta

Por ejemplo, aquí hay una pequeña sección de las rutas de Rails de salida para una ruta `RESTful`:

```ruby
    users GET    /users(.:format)          users#index
          POST   /users(.:format)          users#create
 new_user GET    /users/new(.:format)      users#new
edit_user GET    /users/:id/edit(.:format) users#edit
```

Puede buscar en sus rutas con la opción grep: `-g`. Esto genera cualquier ruta que coincida parcialmente con el nombre del método de ayuda de URL, el verbo `HTTP` o la ruta de acceso de URL.

```ruby
$ bin/rails routes -g new_comment
$ bin/rails routes -g POST
$ bin/rails routes -g admin
```

Si sólo desea ver las rutas que se asignan a un controlador específico, existe la opción `-c`.

```ruby
$ bin/rails routes -c users
$ bin/rails routes -c admin/users
$ bin/rails routes -c Comments
$ bin/rails routes -c Articles::CommentsController
```

> Usted encontrará que la salida de las rutas de Rails son mucho más legibles si usted ensancha su ventana terminal hasta que las líneas de la salida no envuelvan.

### 5.2 Pruebas de rutas

Las rutas deben incluirse en su estrategia de testing \(al igual que el resto de su aplicación\). Rails ofrece tres aserciones integradas diseñadas para simplificar las pruebas de las rutas:

* assert\_generates
* assert\_recognizes
* assert\_routing

#### 5.2.1 La aserción assert\_generates

`Assert_generates` afirma que un determinado conjunto de opciones genera una ruta determinada y puede utilizarse con rutas predeterminadas o rutas personalizadas. Por ejemplo:

```ruby
assert_generates '/photos/1', { controller: 'photos', action: 'show', id: '1' }
assert_generates '/about', controller: 'pages', action: 'about'
```

#### 5.2.2 La aserción assert\_recognizes 

`Assert_recognizes` es la inversa de `assert_generates`. Afirma que un camino determinado es reconocido y lo encamina a un lugar particular de su aplicación. Por ejemplo:

```ruby
assert_recognizes({ controller: 'photos', action: 'show', id: '1' }, '/photos/1')
```

Puede proporcionar un argumento `:method` para especificar el verbo `HTTP`:

```ruby
assert_recognizes({ controller: 'photos', action: 'create' }, { path: 'photos', method: :post })
```

#### 5.2.3 Aserción assert\_routing

La aserción `assert_routing` comprueba la ruta en ambos sentidos: prueba que la ruta genera las opciones y que las opciones generan la ruta. Así, combina las funciones de `assert_generates` y `assert_recognizes`:

```ruby
assert_routing({ path: 'photos', method: :post }, { controller: 'photos', action: 'create' })
```



