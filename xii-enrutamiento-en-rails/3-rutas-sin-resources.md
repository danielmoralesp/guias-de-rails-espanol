# 3- Rutas sin resources

Además del enrutamiento por resources, Rails tiene un poderoso apoyo para el enrutamiento de direcciones URL arbitrarias a las acciones. Aquí, no obtiene grupos de rutas generadas automáticamente por resources. En su lugar, configura cada ruta de su aplicación por separado.

Mientras que normalmente debe usar resources, todavía hay muchos lugares donde el enrutamiento simple es más apropiado. No hay necesidad de intentar cazar cada pieza de su aplicación en un marco de recursos si no es un buen ajuste.

En particular, el enrutamiento sencillo facilita el mapeo de las URL heredadas a las nuevas acciones de Rails.

### 3.1 Parámetros vinculados

Cuando configura una ruta regular, proporciona una serie de símbolos que Rails asigna a partes de una solicitud `HTTP` entrante. Por ejemplo, considere esta ruta:

```ruby
get 'photos(/:id)', to: :display
```

Si una solicitud entrante a `/photos/1` es procesada por esta ruta \(porque no ha coincidido con ninguna ruta anterior en el archivo\), entonces el resultado será invocar la acción de visualización del `PhotosController` y hacer que el parámetro final "`1`" este disponible como `params[:id]`. Esta ruta también encaminará la solicitud entrante de `/photos` a `PhotosController#display`, ya que el `:id` es un parámetro opcional, indicado por paréntesis.

### 3.2 Segmentos dinámicos

Puede configurar tantos segmentos dinámicos dentro de una ruta regular como desee. Cualquier segmento estará disponible para la acción como parte de `params`. Si configura esta ruta:

```ruby
get 'photos/:id/:user_id', to: 'photos#show'
```

Una ruta entrante de `/photos/1/2` será enviada a la acción `show` del `PhotosController`. `params[: id]` será "`1`", y `params[:user_id]` será "`2`".

> De forma predeterminada, los segmentos dinámicos no aceptan puntos, ya que el punto se utiliza como separador para las rutas formateadas. Si necesita usar un punto dentro de un segmento dinámico, añada una restricción que reemplace esto: por ejemplo, `id: /[^\/]+/` permite cualquier cosa menos una barra.

### 3.3 Segmentos estáticos

Puede especificar segmentos estáticos al crear una ruta, no añadiendo dos puntos a un fragmento:

```ruby
get 'photos/:id/with_user/:user_id', to: 'photos#show'
```

Esta ruta respondería a rutas como `/photos/1/with_user/2`. En este caso, `params` sería `{ controller: 'photos', action: 'show', id: '1', user_id: '2' }`.

### 3.4 Query string

Los parámetros también incluirán cualquier parámetro de la cadena de consulta. Por ejemplo, con esta ruta:

```ruby
get 'photos/:id', to: 'photos#show'
```

Una ruta entrante de `/photos/1?user_id=2` será enviada a la acción show del controlador `Photos`. `params` será `{ controller: 'photos', action: 'show', id: '1', user_id: '2' }`.

### 3.5 Definición de valores predeterminados

Puede definir valores predeterminados en una ruta suministrando un hash para la opción `:defaults`. Esto incluso se aplica a los parámetros que no especifica como segmentos dinámicos. Por ejemplo:

```ruby
get 'photos/:id', to: 'photos#show', defaults: { format: 'jpg' }
```

Rails coincidiría con las `photos/12` con la acción `show` de `PhotosController`, y establecería `params[:format]` a "`jpg`".

También puede utilizar valores predeterminados en un formato de bloque para definir los valores predeterminados para varios elementos:

```ruby
defaults format: :json do
  resources :photos
end
```

> No puede anular los valores predeterminados a través de parámetros de consulta, por razones de seguridad. Los únicos valores por defecto que se pueden sobreescribir son los segmentos dinámicos a través de la sustitución en la ruta de la URL.

### 3.6 Nombrar las rutas

Puede especificar un nombre para cualquier ruta utilizando la opción `:as`:

```ruby
get 'exit', to: 'sessions#destroy', as: :logout
```

Esto creará `logout_path` y `logout_url` como ayudantes nombrados en su aplicación. Llamando a `logout_path` retornará `/exit`

También puede utilizar esta opción para anular métodos de enrutamiento definidos por recursos, como estos:

```ruby
get ':username', to: 'users#show', as: :user
```

Esto definirá un método `user_path` que estará disponible en los controladores, ayudantes y vistas que irán a una ruta como `/bob`. Dentro de la acción `show` de `UsersController`, `params[:username]` contendrá el nombre de usuario para el usuario. cambie `:username` en la definición de ruta si no desea que su nombre de parámetro sea `:username`.







