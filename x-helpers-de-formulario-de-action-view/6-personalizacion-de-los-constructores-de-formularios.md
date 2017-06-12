# 6- Personalización de los constructores de formularios

Como se mencionó anteriormente, el objeto generado por `form_for` y `fields_for` es una instancia de `FormBuilder` \(o una subclase del mismo\). Los constructores de formularios encapsulan la noción de mostrar elementos de formulario para un solo objeto. Mientras que usted puede, por supuesto, escribir ayudantes para sus formularios de la manera habitual, también puede usar la subclase `FormBuilder` y agregar los ayudantes allí. Por ejemplo:

```ruby
<%= form_for @person do |f| %>
  <%= text_field_with_label f, :first_name %>
<% end %>
```

Puede ser reemplazado por

```ruby
<%= form_for @person, builder: LabellingFormBuilder do |f| %>
  <%= f.text_field :first_name %>
<% end %>
```

Definiendo una clase `LabellingFormBuilder` similar a la siguiente:

```ruby
class LabellingFormBuilder < ActionView::Helpers::FormBuilder
  def text_field(attribute, options={})
    label(attribute) + super
  end
end
```

Si reutiliza esto con frecuencia, puede definir un ayudante `label_form_for` que aplique automáticamente al constructor: por ejemplo la Opción `LabellingFormBuilder`:

```ruby
def labeled_form_for(record, options = {}, &block)
  options.merge! builder: LabellingFormBuilder
  form_for record, options, &block
end
```

El constructor de formularios utilizado también determina qué sucede cuando lo hace:

```ruby
<%= render partial: f %>
```

Si `f` es una instancia de `FormBuilder`, esto hará que el formulario sea parcial, estableciendo el objeto parcial en el constructor de formularios. Si el constructor de formularios es de la clase `LabellingFormBuilder`, entonces se renderizará la parte de `labelling_form` en su lugar.

