# 1- El Propósito del Enrutador de Rails

El router de Rails reconoce las URL y las envía a la acción de un controlador, o a una aplicación de Rack. También puede generar rutas de acceso y direcciones URL, evitando la necesidad de codificar las cadenas en sus vistas.

### 1.1 Conexión de URLs al código

Cuando su aplicación Rails recibe una solicitud de entrada para:

```ruby
GET /patients/17
```

pide al router que coincida con una acción del controlador. Si la primera ruta coincidente es:

```ruby
get '/patients/:id', to: 'patients#show'
```

La solicitud se envía al controlador `patients` de la accion `show` `{id: '17'}` en params.

### 1.2 Generación de rutas y URL desde el código

También puede generar rutas de acceso y URL. Si la ruta de arriba es modificada para ser:

```ruby
get '/patients/:id', to: 'patients#show', as: 'patient'
```

Y su aplicación contiene este código en el controlador:

```ruby
@patient = Patient.find(17)
```

Y esto en la vista correspondiente:

```ruby
<%= link_to 'Patient Record', patient_path(@patient) %>
```

Entonces el enrutador generará el path `/patients/17`. Esto reduce la fragilidad de su vista y hace que su código sea más fácil de entender. Tenga en cuenta que no es necesario especificar el `ID` en el ayudante de ruta.

