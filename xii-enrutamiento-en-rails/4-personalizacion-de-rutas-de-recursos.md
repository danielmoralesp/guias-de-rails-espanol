# 4- Personalización de rutas de recursos

Mientras que las rutas predeterminadas y los ayudantes generados por `resources: :articles` suelen servirle bien, es posible que desee personalizarlos de alguna manera. Rails le permite personalizar prácticamente cualquier parte genérica de los helpers de recursos.

### 4.1 Especificación de un controlador a utilizar

La opción `:controller` le permite especificar explícitamente un controlador para utilizarlo en el recurso. Por ejemplo:

```ruby
resources :photos, controller: 'images'
```

Reconocerán los paths entrantes que empiezan con `/photos`, pero la ruta al controlador de imágenes:

| HTTP Verb | Path | Controller\#Action | Named Helper |
| :--- | :--- | :--- | :--- |
| GET | /photos | images\#index | photos\_path |
| GET | /photos/new | images\#new | new\_photo\_path |
| POST | /photos | images\#create | photos\_path |
| GET | /photos/:id | images\#show | photo\_path\(:id\) |
| GET | /photos/:id/edit | images\#edit | edit\_photo\_path\(:id\) |
| PATCH/PUT | /photos/:id | images\#update | photo\_path\(:id\) |
| DELETE | /photos/:id | images\#destroy | photo\_path\(:id\) |

> Utilice `photos_path`, `new_photo_path`, etc. para generar rutas de acceso para este recurso.

Para los controladores con espacios de nombres, puede utilizar la notación de directorio. Por ejemplo:

```ruby
resources :user_permissions, controller: 'admin/user_permissions'
```

Esto hará que se dirija al controlador `Admin::UserPermissions`.

> Sólo se admite la notación de directorios. Especificar el controlador con la notación de Constantes de Ruby \(por ejemplo, `controller: 'Admin::UserPermissions'`\) puede conducir a problemas de enrutamiento y los resultados en una advertencia.

### 4.2 Especificación de restricciones

Puede utilizar la opción `:constraints` para especificar un formato requerido en el `id` implícito. Por ejemplo:

```ruby
resources :photos, constraints: { id: /[A-Z][A-Z][0-9]+/ }
```

Esta declaración limita el parámetro `:id` para que coincida con la expresión regular suministrada. Por lo tanto, en este caso, el enrutador ya no coincidiría con esta ruta `/photos/1` . En su lugar coincidiría con,`/photos/RR27` .

Puede especificar una restricción única para aplicar a un número de rutas utilizando el formulario de bloque:

```ruby
constraints(id: /[A-Z][A-Z][0-9]+/) do
  resources :photos
  resources :accounts
end
```

> Por supuesto, puede utilizar las limitaciones más avanzadas disponibles en rutas sin-resources en este contexto.
>
> De forma predeterminada, el parámetro `id` no acepta puntos, ya que el punto se utiliza como separador para las rutas formateadas. Si necesitas usar un punto dentro de un `:id` agrega una restricción que anula esto - por ejemplo `id: /[^\/]+/` permite cualquier cosa menos una barra.

### 4.3 Sobre-escritura de los helpers nombrados

La opción `:as` le permite anular la denominación normal para los ayudantes de ruta designados. Por ejemplo:

```ruby
resources :photos, as: 'images'
```

Reconocerán las rutas entrantes que empiezan con `/photos` y enrutarán las solicitudes a `PhotosController`, pero usarán el valor de la opción `:as` para nombrar a los helpers.

| HTTP Verb | Path | Controller\#Action | Named Helper |
| :--- | :--- | :--- | :--- |
| GET | /photos | photos\#index | images\_path |
| GET | /photos/new | photos\#new | new\_image\_path |
| POST | /photos | photos\#create | images\_path |
| GET | /photos/:id | photos\#show | image\_path\(:id\) |
| GET | /photos/:id/edit | photos\#edit | edit\_image\_path\(:id\) |
| PATCH/PUT | /photos/:id | photos\#update | image\_path\(:id\) |
| DELETE | /photos/:id | photos\#destroy | image\_path\(:id\) |

### 4.4 Sobre-escritura de los segmentos new y de edit

La opción `:path_names` le permite sobre-escribir los segmentos `new` y `edit` automáticamente en las rutas:

```ruby
resources :photos, path_names: { new: 'make', edit: 'change' }
```

Esto haría que el enrutamiento reconozca los paths como:

```ruby
/photos/make
/photos/1/change
```

> Esta opción no cambia los nombres de los action reales. Los dos paths mostrados seguirían la ruta hacia las acciones `new` y de `edit`.
>
> Si desea cambiar esta opción de manera uniforme para todas sus rutas, puede utilizar un scope.

```ruby
scope path_names: { new: 'make' } do
  # rest of your routes
end
```

### 4.5 Prefijo de los ayudantes de ruta designados

Puede utilizar la opción `:as` para prefijar los ayudantes de ruta designados que Rails genera para una ruta. Utilice esta opción para evitar colisiones de nombres entre rutas utilizando un scope de ruta. Por ejemplo:

```ruby
scope 'admin' do
  resources :photos, as: 'admin_photos'
end

resources :photos
```

Esto proporcionará ayudantes de ruta tales como `admin_photos_path`, `new_admin_photo_path`, etc.

Para prefijar un grupo de ayudantes de ruta, utilice `:as` con `scope`:

```ruby
scope 'admin', as: 'admin' do
  resources :photos, :accounts
end
 
resources :photos, :accounts
```

Esto generará rutas como `admin_photos_path` y `admin_accounts_path` que se asignan a `/admin/photos` y `/admin/accounts` respectivamente.

> El scope de namespaces agregará automáticamente :as, así como los prefijos `:module` y `:path`

También puede prefijar rutas con un parámetro con nombre:

```ruby
scope ':username' do
  resources :articles
end
```

Esto le proporcionará URLs como `/bob/articles/1` y le permitirá hacer referencia a la parte del nombre de usuario de la ruta de acceso como `params[: username]` en los controladores, ayudantes y vistas.

### 4.6 Restricción de las rutas creadas

De forma predeterminada, Rails crea rutas para las siete acciones predeterminadas \(`index`, `show`, `new`, `create`, `edit`, `update` y `destroy`\) para cada ruta RESTful en su aplicación. Puede utilizar las opciones `:only` y `:except` para ajustar este comportamiento. La opción `:only` le dice a Rails que cree sólo las rutas especificadas:

```ruby
resources :photos, only: [:index, :show]
```

Ahora, una solicitud `GET` `/photos` tendría éxito, pero una solicitud `POST` `/photos` \(que normalmente se encaminan a la acción `create`\) fallará.

La opción `:except` especifica una ruta o lista de rutas que Rails no debe crear:

```ruby
resources :photos, except: :destroy
```

En este caso, Rails creará todas las rutas normales excepto la ruta para destruir \(una solicitud `DELETE` a `/photos/:id`\).

Si su aplicación tiene muchas rutas RESTful, use `:only` y `:except` para generar sólo las rutas que realmente necesita puede reducir el uso de memoria y acelerar el proceso de enrutamiento.

### 4.7 Paths traducidos

Utilizando el scope, podemos alterar los nombres de ruta generados por los recursos:

```ruby
scope(path_names: { new: 'neu', edit: 'bearbeiten' }) do
  resources :categories, path: 'kategorien'
end
```

Rails ahora crea rutas a `CategoriesController`.

| HTTP Verb | Path | Controller\#Action | Named Helper |
| :--- | :--- | :--- | :--- |
| GET | /kategorien | categories\#index | categories\_path |
| GET | /kategorien/neu | categories\#new | new\_category\_path |
| POST | /kategorien | categories\#create | categories\_path |
| GET | /kategorien/:id | categories\#show | category\_path\(:id\) |
| GET | /kategorien/:id/bearbeiten | categories\#edit | edit\_category\_path\(:id\) |
| PATCH/PUT | /kategorien/:id | categories\#update | category\_path\(:id\) |
| DELETE | /kategorien/:id | categories\#destroy | category\_path\(:id\) |

### 4.8 Sobre-escribir la forma singular

Si desea definir la forma singular de un recurso, debe agregar reglas adicionales al Inflector:

```ruby
ActiveSupport::Inflector.inflections do |inflect|
  inflect.irregular 'tooth', 'teeth'
end
```

### 4.9 Uso de :as en recursos anidados

La opción `:as` reemplaza al nombre generado automáticamente para el recurso en ayudantes de rutas anidadas. Por ejemplo:

```ruby
resources :magazines do
  resources :ads, as: 'periodical_ads'
end
```

Esto creará ayudantes de enrutamiento como `magazine_periodical_ads_url` y `edit_magazine_periodical_ad_path`.

### 4.10 Sobre-escritura de los parámetros de ruta con nombre

La opción `:param` reemplaza al identificador de recurso predeterminado `:id` \(nombre del segmento dinámico utilizado para generar las rutas\). Puede acceder a ese segmento desde su controlador usando `params[<: param>]`.

```ruby
resources :videos, param: :identifier
```

```ruby
     videos GET  /videos(.:format)                  videos#index
            POST /videos(.:format)                  videos#create
 new_videos GET  /videos/new(.:format)              videos#new
edit_videos GET  /videos/:identifier/edit(.:format) videos#edit
```

```ruby
Video.find_by(identifier: params[:identifier])
```

Puede reemplazar `ActiveRecord::Base#to_param` de un modelo relacionado para construir una URL:

```ruby
class Video < ApplicationRecord
  def to_param
    identifier
  end
end
 
video = Video.find_by(identifier: "Roman-Holiday")
edit_videos_path(video) # => "/videos/Roman-Holiday"
```



