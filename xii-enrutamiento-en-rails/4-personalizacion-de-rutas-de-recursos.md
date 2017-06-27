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

Esta declaración limita el parámetro `:id` para que coincida con la expresión regular suministrada. Por lo tanto, en este caso, el enrutador ya no coincidiría con esta ruta `/photos/1` . En su lugar coincidiría con,` /photos/RR27` .

Puede especificar una restricción única para aplicar a un número de rutas utilizando el formulario de bloque:

```ruby
constraints(id: /[A-Z][A-Z][0-9]+/) do
  resources :photos
  resources :accounts
end
```

> Por supuesto, puede utilizar las limitaciones más avanzadas disponibles en rutas sin-resources en este contexto.

> De forma predeterminada, el parámetro `id` no acepta puntos, ya que el punto se utiliza como separador para las rutas formateadas. Si necesitas usar un punto dentro de un `:id` agrega una restricción que anula esto - por ejemplo `id: /[^\/]+/` permite cualquier cosa menos una barra.



