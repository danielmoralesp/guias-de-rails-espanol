# 2- Ruta resource: el valor predeterminado de Rails

El enrutamiento de recursos le permite declarar rápidamente todas las rutas comunes para un controlador con recursos. En lugar de declarar rutas separadas para sus acciones `index`, `show`, `new`, `edit`, `create`, `update` y `destroy` , una ruta de recursos las declara en una sola línea de código.

### 2.1 Resources en la Web

Los navegadores solicitan páginas desde Rails realizando una solicitud de `URL` utilizando un método `HTTP` específico, como `GET`, `POST`, `PATCH`, `PUT` y `DELETE`. Cada método es una solicitud para realizar una operación en el recurso. Una ruta de recursos traza un número de solicitudes relacionadas a acciones en un solo controlador.

Cuando su aplicación Rails recibe una solicitud de entrada para:

```ruby
DELETE /photos/17
```

Pide al enrutador que lo mande a una acción del controlador. Si la primera ruta coincidente es:

```ruby
resources :photos
```

Rails enviará esa solicitud a la acción `destroy` en el controlador de fotos con `{id: '17'}` en params.

### 2.2 CRUD, verbos y acciones

En Rails, una ruta de recursos proporciona una correlación entre los verbos `HTTP` y las `URL` a las acciones del controlador. Por convención, cada acción también se correlaciona con una operación `CRUD` específica en una base de datos. Una sola entrada en el archivo de enrutamiento, como por ejemplo:

```ruby
resources :photos
```

crea siete rutas diferentes en su aplicación, todas mapeando al controlador de fotos:

| HTTP Verb | Path | Controller\#Action | Used for |
| :--- | :--- | :--- | :--- |
| GET | /photos | photos\#index | muestra una lista de todas las fotos |
| GET | /photos/new | photos\#new | devuelve un formulario HTML para crear una nueva foto |
| POST | /photos | photos\#create | crea una nueva foto |
| GET | /photos/:id | photos\#show | muestra una foto en especifico |
| GET | /photos/:id/edit | photos\#edit | retorna un formulario HTML para editar la foto |
| PATCH/PUT | /photos/:id | photos\#update | actualiza una foto en especifico |
| DELETE | /photos/:id | photos\#destroy | elimina una foto en especifico |

> Debido a que el enrutador usa el verbo `HTTP` y la `URL` para coincidir con las solicitudes entrantes, cuatro direcciones URL asignan siete acciones diferentes.
>
> Las rutas de Rails se emparejan en el orden en que se especifican, por lo que si tienes `resources :photos`  arriba de `get 'photos/poll'` la ruta de la acción  `show` para la línea de `resources` coincidirá antes de la línea `get`. Para arreglar esto, mueva la línea `get` por encima de la línea de `resources` para que coincida primero.

### 2.3 Ayudantes de ruta y URL

La creación de una ruta `resources` también expondrá una serie de ayudantes a los controladores en su aplicación. En el caso de `resources :photos`

* `photos_path`retorna`/photos`
* `new_photo_path`retorna`/photos/new`
* `edit_photo_path(:id)`retorna`/photos/:id/edit` \(por ejemplo, `edit_photo_path(10)` retorna `/photos/10/edit`\)
* `photo_path(:id)`retorna`/photos/:id` \(por ejemplo, `photo_path(10) retorna` `/photos/10`\)

Cada uno de estos ayudantes tiene un ayudante `_url` correspondiente \(como `photos_url`\) que devuelve la misma ruta prefijada con el prefijo actual de host, puerto y ruta.

### 2.4 Definir varios recursos al mismo tiempo

Si necesita crear rutas para más de un recurso, puede guardar un poco de escritura definiéndolas todas con una sola llamada a `resources`:

```ruby
resources :photos, :books, :videos
```

Esto funciona exactamente igual que:

```ruby
resources :photos
resources :books
resources :videos
```

### 2.5 Recursos Singulares

A veces, tiene un recurso que los clientes siempre buscan sin hacer referencia a un `ID`. Por ejemplo, desea que `/profile` muestre siempre el perfil del usuario actualmente conectado. En este caso, puede utilizar un recurso singular para asignar `/profile` \(en lugar de `/profile/:id`\) a la acción `show`:

```ruby
get 'profile', to: 'users#show'
```

Pasar un string a `get` esperará un formato `controller#action`, y al pasar un símbolo se asignará directamente a una acción, pero también debe especificar el `controller:` a utilizar:

```ruby
get 'profile', to: :show, controller: 'users'
```

Esta ruta resourceful \(en singular\):

```ruby
resource :geocoder
```

Crea seis rutas diferentes en su aplicación, todas mapeadas al controlador `Geocoders`:

| HTTP Verb | Path | Controller\#Action | Used for |
| :--- | :--- | :--- | :--- |
| GET | /geocoder/new | geocoders\#new | retorna un formulario  HTML para crear el geocoder |
| POST | /geocoder | geocoders\#create | crea el nuevo geocoder |
| GET | /geocoder | geocoders\#show | muestra el único recurso de geocoder |
| GET | /geocoder/edit | geocoders\#edit | devuelve un formulario HTML para editar el geocoder |
| PATCH/PUT | /geocoder | geocoders\#update | actualiza el único recurso de geocoder |
| DELETE | /geocoder | geocoders\#destroy | elimina el recurso geocoder |

Como es posible que desee utilizar el mismo controlador para una ruta singular \(`/account`\) y una ruta plural \(`/accounts/45`\), los recursos singulares se asignan a controladores múltiples. Por ejemplo, `resource: photo` y `resources: photos` crea rutas singulares y plurales que se asignan al mismo controlador \(`PhotosController`\).

Una ruta singular genera estos ayudantes:

* `new_geocoder_path`retorna`/geocoder/new`
* `edit_geocoder_path`retorna`/geocoder/edit`
* `geocoder_path`retorna`/geocoder`

Al igual que con recursos en plural, los mismos helpers que terminan en `_url` también incluirán el prefijo host, port y path.

Un error de larga data impide que `form_for` trabaje automáticamente con recursos singulares. Como solución, especifique la URL del formulario directamente, de la siguiente manera:

```ruby
form_for @geocoder, url: geocoder_path do |f|

# snippet for brevity
```

### 2.6 Namespaces del controlador y enrutamiento

Es posible que desee organizar grupos de controladores en un espacio de nombres. Más comúnmente, puede agrupar varios controladores administrativos bajo un `Admin::` namespace. Usted puede colocar estos controladores en el directorio `app/controllers/admin`, y puede agruparlos en su enrutador:

```ruby
namespace :admin do
  resources :articles, :comments
end
```

Esto creará una serie de rutas para cada uno de los controladores `articles` y `comments` . Para `Admin::ArticlesController`, Rails creará:

| HTTP Verb | Path | Controller\#Action | Named Helper |
| :--- | :--- | :--- | :--- |
| GET | /admin/articles | admin/articles\#index | admin\_articles\_path |
| GET | /admin/articles/new | admin/articles\#new | new\_admin\_article\_path |
| POST | /admin/articles | admin/articles\#create | admin\_articles\_path |
| GET | /admin/articles/:id | admin/articles\#show | admin\_article\_path\(:id\) |
| GET | /admin/articles/:id/edit | admin/articles\#edit | edit\_admin\_article\_path\(:id\) |
| PATCH/PUT | /admin/articles/:id | admin/articles\#update | admin\_article\_path\(:id\) |
| DELETE | /admin/articles/:id | admin/articles\#destroy | admin\_article\_path\(:id\) |

Si desea enrutar `/articles` \(sin el prefijo `/admin`\) a `Admin::ArticlesController`, puede utilizar:

```ruby
scope module: 'admin' do
  resources :articles, :comments
end
```

O, en un solo caso:

```ruby
resources :articles, module: 'admin'
```

Si desea enrutar `/admin/artículos` a `ArticlesController` \(sin el prefijo `Admin::module`\), puede utilizar:

```ruby
scope '/admin' do
  resources :articles, :comments
end
```

O, en un solo caso:

```ruby
resources :articles, path: '/admin/articles'
```

En cada uno de estos casos, las rutas nombradas siguen siendo las mismas que si no utiliza el ámbito. En el último caso, las siguientes rutas mapean a `ArticlesController`:

| HTTP Verb | Path | Controller\#Action | Named Helper |
| :--- | :--- | :--- | :--- |
| GET | /admin/articles | articles\#index | articles\_path |
| GET | /admin/articles/new | articles\#new | new\_article\_path |
| POST | /admin/articles | articles\#create | articles\_path |
| GET | /admin/articles/:id | articles\#show | article\_path\(:id\) |
| GET | /admin/articles/:id/edit | articles\#edit | edit\_article\_path\(:id\) |
| PATCH/PUT | /admin/articles/:id | articles\#update | article\_path\(:id\) |
| DELETE | /admin/articles/:id | articles\#destroy | article\_path\(:id\) |

Si necesita usar un espacio de nombres de controlador diferente dentro de un bloque de espacio de nombres, puede especificar una ruta de controlador absoluta, por ejemplo:  `get '/foo' => '/foo#index`''.

### 2.7 Recursos anidados

Es común tener recursos que son lógicamente hijos de otros recursos. Por ejemplo, suponga que su aplicación incluye estos modelos:

```ruby
class Magazine < ApplicationRecord
  has_many :ads
end

class Ad < ApplicationRecord
  belongs_to :magazine
end
```

Las rutas anidadas permiten capturar esta relación en el enrutamiento. En este caso, puede incluir esta declaración de ruta:

```ruby
resources :magazines do
  resources :ads
end
```

Además de las rutas para magazines, esta declaración también enruta los anuncios a un `AdsController.` Las URL de los ads requieren un magazine:

| HTTP Verb | Path | Controller\#Action | Used for |
| :--- | :--- | :--- | :--- |
| GET | /magazines/:magazine\_id/ads | ads\#index | muestra una lista de todos los anuncios de una revista específica |
| GET | /magazines/:magazine\_id/ads/new | ads\#new | devuelve un formulario HTML para crear un nuevo anuncio perteneciente a una revista específica |
| POST | /magazines/:magazine\_id/ads | ads\#create | crea un nuevo anuncio perteneciente a una revista específica |
| GET | /magazines/:magazine\_id/ads/:id | ads\#show | muestra un anuncio específico perteneciente a una revista específica |
| GET | /magazines/:magazine\_id/ads/:id/edit | ads\#edit | devuelve un formulario HTML para editar un anuncio perteneciente a una revista específica |
| PATCH/PUT | /magazines/:magazine\_id/ads/:id | ads\#update | actualiza un anuncio específico perteneciente a una revista específica |
| DELETE | /magazines/:magazine\_id/ads/:id | ads\#destroy | elimina un anuncio específico perteneciente a una revista específica |

Esto también creará ayudantes de enrutamiento como `magazine_ads_url` y `edit_magazine_ad_path`. Estos ayudantes toman una instancia de magazine como el primer parámetro \(`magazine_ads_url(@magazine)`\).

#### 2.7.1 Límites de anidamiento

Puede anidar recursos dentro de otros recursos anidados si lo desea. Por ejemplo:

```ruby
resources :publishers do
  resources :magazines do
    resources :photos
  end
end
```

Los recursos profundamente anidados se vuelven rápidamente engorrosos. En este caso, por ejemplo, la aplicación reconocería rutas tales como:

```ruby
/publishers/1/magazines/2/photos/3
```

El auxiliar de ruta correspondiente sería `publisher_magazine_photo_url`, lo que requiere que especifique objetos en los tres niveles. De hecho, esta situación es bastante confusa que un artículo popular de Jamis Buck propone una regla de oro para el buen diseño de Rails:

> Los recursos nunca deben anidarse más de 1 nivel de profundidad.

#### 2.7.2 Anidación poco profunda

Una manera de evitar la anidación profunda \(como se recomienda anteriormente\) es generar una coleccion de acciones dentro de un ambito del pariente, para obtener un sentido de la jerarquía, pero no anidar las acciones de los miembros. En otras palabras, sólo para construir rutas con la cantidad mínima de información para identificar de forma exclusiva el recurso, como esto:

```ruby
resources :articles do
  resources :comments, only: [:index, :new, :create]
end
resources :comments, only: [:show, :edit, :update, :destroy]
```

Esta idea establece un equilibrio entre las rutas descriptivas y el anidamiento profundo. Existe sintaxis abreviada para lograr eso, a través de la opción `:shallow`:

```ruby
resources :articles do
  resources :comments, shallow: true
end
```

Esto generará exactamente las mismas rutas que el primer ejemplo. También puede especificar la opción `:shallow` en el recurso padre, en cuyo caso todos los recursos anidados serán superficiales \(`shallow`\):

```ruby
resources :articles, shallow: true do
  resources :comments
  resources :quotes
  resources :drafts
end
```

El método superficial \(`shallow`\) de la DSL crea un ámbito dentro del cual cada anidado es superficial. Esto genera las mismas rutas que el ejemplo anterior:

```ruby
shallow do
  resources :articles do
    resources :comments
    resources :quotes
    resources :drafts
  end
end
```

Existen dos opciones de ámbito para personalizar rutas poco profundas. prefijos `:shallow_path`  del path miembro con el parámetro especificado:

```ruby
scope shallow_path: "sekret" do
  resources :articles do
    resources :comments, shallow: true
  end
end
```

El recurso de comentarios aquí tendrá las siguientes rutas generadas:

| HTTP Verb | Path | Controller\#Action | Named Helper |
| :--- | :--- | :--- | :--- |
| GET | /articles/:article\_id/comments\(.:format\) | comments\#index | article\_comments\_path |
| POST | /articles/:article\_id/comments\(.:format\) | comments\#create | article\_comments\_path |
| GET | /articles/:article\_id/comments/new\(.:format\) | comments\#new | new\_article\_comment\_path |
| GET | /sekret/comments/:id/edit\(.:format\) | comments\#edit | edit\_comment\_path |
| GET | /sekret/comments/:id\(.:format\) | comments\#show | comment\_path |
| PATCH/PUT | /sekret/comments/:id\(.:format\) | comments\#update | comment\_path |
| DELETE | /sekret/comments/:id\(.:format\) | comments\#destroy | comment\_path |

La opción: `shallow_prefix` añade el parámetro especificado a los ayudantes nombrados:

```ruby
scope shallow_prefix: "sekret" do
  resources :articles do
    resources :comments, shallow: true
  end
end
```

El recurso `comments` aquí tendrá las siguientes rutas generadas para él:

| HTTP Verb | Path | Controller\#Action | Named Helper |
| :--- | :--- | :--- | :--- |
| GET | /articles/:article\_id/comments\(.:format\) | comments\#index | article\_comments\_path |
| POST | /articles/:article\_id/comments\(.:format\) | comments\#create | article\_comments\_path |
| GET | /articles/:article\_id/comments/new\(.:format\) | comments\#new | new\_article\_comment\_path |
| GET | /comments/:id/edit\(.:format\) | comments\#edit | edit\_sekret\_comment\_path |
| GET | /comments/:id\(.:format\) | comments\#show | sekret\_comment\_path |
| PATCH/PUT | /comments/:id\(.:format\) | comments\#update | sekret\_comment\_path |
| DELETE | /comments/:id\(.:format\) | comments\#destroy | sekret\_comment\_path |

### 2.8 Routing concerns

El Routing concerns le permite declarar rutas comunes que pueden reutilizarse dentro de otros recursos y rutas. Para definir un concerns:

```ruby
concern :commentable do
  resources :comments
end

concern :image_attachable do
  resources :images, only: :index
end
```

Estos concerns pueden utilizarse en recursos para evitar la duplicación de código y compartir el comportamiento entre rutas:

```ruby
resources :messages, concerns: :commentable

resources :articles, concerns: [:commentable, :image_attachable]
```

Lo anterior es equivalente a:

```ruby
resources :messages do
  resources :comments
end

resources :articles do
  resources :comments
  resources :images, only: :index
end
```

También puedes usarlos en cualquier lugar que quieras dentro de las rutas, por ejemplo en una llamada de ámbito o de espacio de nombres:

```ruby
namespace :articles do
  concerns :commentable
end
```



