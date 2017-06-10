# 2- Tratamiento de objetos del modelo

### 2.1 Helper de objetos del modelo

Una tarea particularmente común para un formulario es editar o crear un objeto del modelo. Mientras que los helper de la forma  `* _tag` se pueden utilizar ciertamente para esta tarea son algo verbosos como para que cada etiqueta que usted necesite, tenga que asegurarse que el nombre correcto del parámetro se esta utilizando y fijar el valor por defecto del `input` apropiadamente. Rails proporciona helpers adaptados para esta tarea. Estos helpers carecen del sufijo `_tag`, por ejemplo `text_field`, `text_area`.

Para estos helpers el primer argumento es el nombre de una variable de instancia y el segundo es el nombre de un método \(generalmente un atributo\) para llamar a ese objeto. Rails establecerá el valor del control del `input` en el valor de retorno de ese método para el objeto y establecerá un nombre de `input` adecuado. Si en su controlador ha definido `@person` y el `name` de esa persona es Henry, entonces el formulario contiene:

```ruby
<%= text_field(:person, :name) %>
```

Producirá una salida similar a:

```ruby
<input id="person_name" name="person[name]" type="text" value="Henry"/>
```

Tras el envío del formulario, el valor ingresado por el usuario se almacenará en `params[:person][:name]`. El hash `params[:person]` es adecuado para pasar a `Person.new` o, si `@person` es una instancia de `Person`, `@person.update`. Mientras que el nombre de un atributo es el segundo parámetro más común, para estos helpers esto no es obligatorio. En el ejemplo anterior, siempre que los objetos persona tengan un `name` y un `name=` a un método Rails estará feliz.

Debe pasar el nombre de una variable de instancia, es decir `:person` o "`person`", no una instancia real del objeto de modelo.

Rails proporciona ayuda para mostrar los errores de validación asociados con un objeto del modelo. Estas son cubiertas en detalle por la guía de validación de Active Record.

### 2.2 Vinculación de un formulario a un objeto

Mientras que esto es un aumento en la comodidad, está lejos de ser perfecto. Si `Person` tiene muchos atributos para editar entonces estaríamos repitiendo el nombre del objeto editado muchas veces. Lo que queremos hacer de alguna manera es vincular un formulario a un objeto del modelo, que es exactamente lo que `form_for` hace.

Supongamos que tenemos un controlador para tratar artículos `app/controllers/articles_controller.rb`:

```ruby
def new
  @article = Article.new
end
```

La vista correspondiente es `app/views/articles/new.html.erb` y que utiliza `form_for` se parece a esto:

```ruby
<%= form_for @article, url: {action: "create"}, html: {class: "nifty_form"} do |f| %>
  <%= f.text_field :title %>
  <%= f.text_area :body, size: "60x12" %>
  <%= f.submit "Create" %>
<% end %>
```

* `@article` es el objeto real que se está editando.
* Hay un solo hash de opciones. Las opciones de enrutamiento se pasan en un hash de `:url`, y las opciones `HTML` se pasan en el hash `:html`. También puede proporcionar una opción de `namespace` para su formulario para garantizar la unicidad de los atributos de `id` en los elementos del formulario. El atributo spacename será prefijado con subrayado en el `id` del HTML generado.
* El método `form_for` produce un objeto form builder \(la variable `f`\).
* Los métodos para crear controles del formulario se llaman en el objeto de constructor de formularios `f`.

El HTML resultante es:

```ruby
<form accept-charset="UTF-8" action="/articles" method="post" class="nifty_form">
  <input id="article_title" name="article[title]" type="text" />
  <textarea id="article_body" name="article[body]" cols="60" rows="12"></textarea>
  <input name="commit" type="submit" value="Create" />
</form>
```

El nombre pasado a `form_for` controla la clave utilizada en `params` para acceder a los valores del formulario. Aquí el nombre es `article` y por lo tanto todos los inputs tienen nombres del formulario `article[attribute_name]`. En consecuencia, en la acción de creación `params[:article]` será un hash con keys `:title` y `:body`. Puede leer más sobre la importancia de los nombres de entrada en la sección `parameter_names`.

Los métodos auxiliares llamados al constructor \(builder\) de formularios son idénticos a los helpers de objetos del modelo, excepto que no es necesario especificar qué objeto se está editando ya que éste ya está administrado por el constructor de formularios.

Puede crear un enlace similar sin crear realmente etiquetas `<form>` con el campo `fields_for` helper. Esto es útil para editar objetos de modelo adicionales con el mismo formulario. Por ejemplo, si tuviera un modelo `Person` con un modelo `ContactDetail` asociado, podría crear un formulario para crear ambos de la siguiente manera:

```ruby
<%= form_for @person, url: {action: "create"} do |person_form| %>
  <%= person_form.text_field :name %>
  <%= fields_for @person.contact_detail do |contact_detail_form| %>
    <%= contact_detail_form.text_field :phone_number %>
  <% end %>
<% end %>
```

Que produce la siguiente salida:

```ruby
<form accept-charset="UTF-8" action="/people" class="new_person" id="new_person" method="post">
  <input id="person_name" name="person[name]" type="text" />
  <input id="contact_detail_phone_number" name="contact_detail[phone_number]" type="text" />
</form>
```

El objeto dado por `fields_for` es un constructor de formularios como el que se obtiene por `form_for` \(de hecho `form_for` llama `fields_for` internamente\).

### 2.3 Basándose en la identificación de registros

El modelo `Article` está directamente disponible para los usuarios de la aplicación, por lo que - siguiendo las mejores prácticas para desarrollar con Rails - debe declararle un `resource`:

```ruby
resources :articles
```

> Declarar un recurso tiene varios efectos secundarios. Vea "Rails Routing From Outside In" para obtener más información sobre cómo configurar y usar recursos.

Cuando se trata de recursos `RESTful`, las llamadas a `form_for` pueden ser significativamente más fáciles si confías en la identificación de registros. En resumen, puede pasar la instancia del modelo y hacer que Rails averigüe el nombre del modelo y el resto:

```ruby
## Creating a new article
# long-style:
form_for(@article, url: articles_path)
# same thing, short-style (record identification gets used):
form_for(@article)

## Editing an existing article
# long-style:
form_for(@article, url: article_path(@article), html: {method: "patch"})
# short-style:
form_for(@article)
```

Observe cómo la invocación de `form_for` de estilo corto es convenientemente la misma, independientemente de que el registro sea nuevo o existente. La identificación de registros es lo suficientemente inteligente como para averiguar si el registro es nuevo,  solicitando `record.new_record?`. También selecciona la ruta correcta para enviar y el nombre basado en la clase del objeto.

Rails también establecerá automáticamente la clase y el `id` de forma apropiada: un formulario que crea un artículo tendría `id` y class `new_article`. Si estaba editando el artículo con `id` 23, la clase se establecería en `edit_article` y en el `id` para `edit_article_23`. Estos atributos se omitirán por brevedad en el resto de esta guía.

> Cuando está utilizando STI \(herencia de tabla única\) con sus modelos, no puede confiar en la identificación de registros en una subclase, si sólo su clase principal se declara a un recurso. Deberá especificar el nombre del modelo, `:url`, y `:method` explicitamente.

#### 2.3.1 Tratamiento de los namespaces

Si ha creado rutas con espacios de nombres, `form_for` tiene una taquigrafía útil para eso también. Si su aplicación tiene un namespace `admin` entonces

```ruby
form_for [:admin, @article]
```

Creará un formulario que se envía al `ArticlesController` dentro del espacio de nombres `admin` \(enviando a `admin_article_path(@article)` en el caso de un `update`\). Si tiene varios niveles de namespacing entonces la sintaxis es similar:

```ruby
form_for [:admin, :management, @article]
```

Para obtener más información sobre el sistema de enrutamiento de Rails y las convenciones asociadas, consulte la guía de enrutamiento.





### 2.4 ¿Cómo funcionan los formularios con métodos PATCH, PUT o DELETE?

El framework de Rails alienta el diseño `RESTful` de sus aplicaciones, lo que significa que va a hacer un montón de solicitudes "`PATCH`" y "`DELETE`" \(además de "`GET`" y "`POST`"\). Sin embargo, la mayoría de los navegadores no admiten métodos distintos de "`GET`" y "`POST`" cuando se trata de enviar formularios.

Rails trabaja alrededor de este problema emulando otros métodos en `POST` con una entrada oculta denominada "`_method`", que se establece para reflejar el método deseado:

```ruby
form_tag(search_path, method: "patch")
```

salida:

```ruby
<form accept-charset="UTF-8" action="/search" method="post">
  <input name="_method" type="hidden" value="patch" />
  <input name="utf8" type="hidden" value="&#x2713;" />
  <input name="authenticity_token" type="hidden" value="f755bb0ed134b76c432144748a6d4b7a7ddf2b71" />
  ...
</form>
```

Al analizar datos POSTed, Rails tendrá en cuenta el parámetro especial `_method`  y actuará como si el método `HTTP` fuera el especificado dentro de él \("`PATCH`" en este ejemplo\).

