# 9- Construcción de formularios complejos

Muchas aplicaciones crecen más allá de formularios simples para editar un solo objeto. Por ejemplo, al crear el objeto `Person`, puede permitir que el usuario \(en el mismo formulario\) cree varios registros de direcciones \(casa, trabajo, etc.\). Cuando más tarde la edición de esa persona el usuario debe ser capaz de agregar, quitar o modificar las direcciones según sea necesario.





### 9.1 Configuración del modelo

Active Record proporciona soporte a nivel de modelo a través del método `accepted_nested_attributes_for`:

```ruby
class Person < ApplicationRecord
  has_many :addresses
  accepts_nested_attributes_for :addresses
end
 
class Address < ApplicationRecord
  belongs_to :person
end
```

Esto crea un método `addresses_attributes=` en `Person` que le permite crear, actualizar y \(opcionalmente\) destruir direcciones.



### 9.2 Formularios anidados

El siguiente formulario le permite al usuario crear un objeto `Person` y sus direcciones asociadas.

```ruby
<%= form_for @person do |f| %>
  Addresses:
  <ul>
    <%= f.fields_for :addresses do |addresses_form| %>
      <li>
        <%= addresses_form.label :kind %>
        <%= addresses_form.text_field :kind %>
 
        <%= addresses_form.label :street %>
        <%= addresses_form.text_field :street %>
        ...
      </li>
    <% end %>
  </ul>
<% end %>
```

Cuando una asociación acepta atributos anidados `fields_for` devuelve su bloque una vez para cada elemento de la asociación. En particular, si una persona no tiene direcciones, no renderiza nada. Un patrón común es que el controlador construya uno o más childrens vacíos de manera que al menos un conjunto de campos se muestre al usuario. El ejemplo siguiente resultaría en 2 conjuntos de campos de dirección que se representan en el formulario de la nueva persona.

```ruby
def new
  @person = Person.new
  2.times { @person.addresses.build}
end
```

El `fields_for` produce un constructor de formularios. El nombre de los parámetros será lo que `accept_nested_attributes_for` espera. Por ejemplo, al crear un usuario con 2 direcciones, los parámetros enviados se verían así:

```ruby
{
  'person' => {
    'name' => 'John Doe',
    'addresses_attributes' => {
      '0' => {
        'kind' => 'Home',
        'street' => '221b Baker Street'
      },
      '1' => {
        'kind' => 'Office',
        'street' => '31 Spooner Street'
      }
    }
  }
}
```

Las claves del hash `:addresses_attributes` no son importantes, solo necesitan ser diferentes para cada dirección.

Si el objeto asociado ya está guardado, `fields_for` genera automáticamente una entrada oculta con el `id` del registro guardado. Puede inhabilitarlo pasando `include_id :false` a `fields_for`. Es posible que desee hacer esto si la entrada generada automáticamente se coloca en una ubicación donde una etiqueta de entrada no es `HTML` válido o cuando se utiliza un ORM donde los children no tienen un `ID`.



### 9.3 El controlador

Como de costumbre, necesitas incluir en la lista blanca los parámetros en el controlador antes de pasarlos al modelo:

```ruby
def create
  @person = Person.new(person_params)
  # ...
end
 
private
  def person_params
    params.require(:person).permit(:name, addresses_attributes: [:id, :kind, :street])
  end
```





### 9.4 Eliminación de objetos

Puede permitir que los usuarios eliminen objetos asociados pasando `allow_destroy :true` a `accept_nested_attributes_for`

```ruby
class Person < ApplicationRecord
  has_many :addresses
  accepts_nested_attributes_for :addresses, allow_destroy: true
end
```

Si el hash de los atributos de un objeto contiene la clave `_destroy` con un valor de `1` o `true` entonces el objeto será destruido. Este formulario permite a los usuarios eliminar direcciones:

```ruby
<%= form_for @person do |f| %>
  Addresses:
  <ul>
    <%= f.fields_for :addresses do |addresses_form| %>
      <li>
        <%= addresses_form.check_box :_destroy%>
        <%= addresses_form.label :kind %>
        <%= addresses_form.text_field :kind %>
        ...
      </li>
    <% end %>
  </ul>
<% end %>
```

No olvide actualizar los parámetros de la lista blanca en su controlador para incluir también el campo `_destroy`:

```ruby
def person_params
  params.require(:person).
    permit(:name, addresses_attributes: [:id, :kind, :street, :_destroy])
end
```





### 9.5 Prevención de registros vacíos

A menudo es útil ignorar conjuntos de campos que el usuario no ha rellenado. Puede controlar esto pasando un `proc: reject_if proc` a `accepted_nested_attributes_for`. Este `proc` será llamado con cada hash de atributos enviados por el formulario. Si el `proc` devuelve `false` entonces Active Record no creará un objeto asociado para ese hash. El ejemplo siguiente sólo intenta generar una dirección si se establece el atributo kind.

```ruby
class Person < ApplicationRecord
  has_many :addresses
  accepts_nested_attributes_for :addresses, reject_if: lambda {|attributes| attributes['kind'].blank?}
end
```

Como conveniencia, puede pasar el símbolo: `all_blank` que creará un proc que rechazará los registros donde todos los atributos están en blanco, excluyendo cualquier valor para `_destroy`.





### 9.6 Adición de campos al vuelo

En lugar de reproducir varios conjuntos de campos con antelación, es posible que desee añadirlos sólo cuando un usuario haga clic en el botón "Agregar nueva dirección". Rails no proporciona ningún soporte incorporado para esto. Al generar nuevos conjuntos de campos debe asegurarse de que la id del array asociado es único: la fecha actual de JavaScript \(milisegundos después del evento\) es una opción común.

