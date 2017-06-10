# 1- Tratamiento de formularios básicos

El helper más basico de los formulario es `form_tag`.

```ruby
<%= form_tag do %>
  Form contents
<% end %>
```

Cuando se llama sin argumentos como éste, crea una etiqueta `<form>` que, cuando se envíe, se publicará en la página actual. Por ejemplo, asumiendo que la página actual es `/home/index`, el `HTML` generado se verá así:

```ruby
<form accept-charset="UTF-8" action="/" method="post">
  <input name="utf8" type="hidden" value="&#x2713;" />
  <input name="authenticity_token" type="hidden" value="J7CBxfHalt49OSHp27hblqK20c9PgwJ108nDHX/8Cts=" />
  Form contents
</form>
```

Observará que el `HTML` contiene un elemento de entrada con el tipo oculto. Esta entrada es importante, porque el formulario no se puede enviar correctamente sin él. El elemento de entrada oculto con el nombre `utf8` hace que los navegadores respeten debidamente la codificación de caracteres de su formulario y se genera para todos los formularios, ya sea que su acción sea "`GET`" o "`POST`".

El segundo elemento de entrada con el nombre `authenticity_token` es una característica de seguridad de Rails llamada "cross-site request forgery protection" \(protección de falsificación de solicitud en el sitio\), y los helpers del formulario lo generan para cada formulario que no sea `GET` \(siempre que esta característica de seguridad esté habilitada\). Puede leer más sobre esto en la Guía de seguridad.

### 1.1 Formulario genérico de búsqueda

Uno de los formularios más básicos que ve en la web es un formulario de búsqueda. Este formulario contiene:

* Un elemento de formulario con el método "`GET`"
* Una etiqueta `input`,
* Un elemento `input text`, y
* Un elemento `submit`.

Para crear este formulario usará `form_tag`, `label_tag`, `text_field_tag` y `submit_tag`, respectivamente. Algo como esto:

```ruby
<%= form_tag("/search", method: "get") do %>
  <%= label_tag(:q, "Search for:") %>
  <%= text_field_tag(:q) %>
  <%= submit_tag("Search") %>
<% end %>
```

Esto generará el siguiente `HTML`:

```ruby
<form accept-charset="UTF-8" action="/search" method="get">
  <input name="utf8" type="hidden" value="&#x2713;" />
  <label for="q">Search for:</label>
  <input id="q" name="q" type="text" />
  <input name="commit" type="submit" value="Search" />
</form>
```

> Para cada `input` del formulario, se genera un atributo `ID` desde su `name` \("`q`" en el ejemplo anterior\). Estos identificadores pueden ser muy útiles para los estilos `CSS` o la manipulación de controles de formulario con `JavaScript`.

Además de `text_field_tag` y `submit_tag`, hay un helper similar para cada control de formulario en HTML.

> Siempre use "`GET`" como el método para los formularios de búsqueda. Esto permite a los usuarios marcar una búsqueda específica y volver a ella. Más generalmente, Rails le anima a usar el verbo `HTTP` correcto para una acción.

### 1.2 Múltiples hashes en las llamadas de helper del formulario

El helper `form_tag` acepta 2 argumentos: la ruta para la acción y un hash de opciones. Este hash especifica el método de envio del formulario y las opciones de `HTML`, como el `class` del elemento de formulario.

```ruby
form_tag(controller: "people", action: "search", method: "get", class: "nifty_form")
# => '<form accept-charset="UTF-8" action="/people/search?method=get&class=nifty_form" method="post">'
```

Aquí, el método y la clase se añaden a la cadena de consulta de la URL generada, ya que aunque usted quiere escribir dos hashes, realmente sólo especificó uno. Así que usted necesita decirle a Ruby lo que, al delimitar el primer hash \(o ambos\) con corchetes. Esto generará el `HTML` que espera:

```ruby
form_tag({controller: "people", action: "search"}, method: "get", class: "nifty_form")
# => '<form accept-charset="UTF-8" action="/people/search" method="get" class="nifty_form">'
```



### 1.3 Helpers para generar elementos del formulario

Rails proporciona una serie de helpers para generar los elementos de un formulario como: casillas de verificación, campos de texto y botones de opción. Estos helpers básicos, con nombres que terminan en `_tag` \(como `text_field_tag` y `check_box_tag`\), generan sólo un elemento `<input>`. El primer parámetro para estos siempre es el nombre del `input`. Cuando se envía el formulario, el nombre se pasará junto con los datos del formulario, y hará su camino a los parámetros en el controlador con el valor introducido por el usuario para ese campo. Por ejemplo, si el formulario contiene `<%= text_field_tag (:query)%>`, entonces podrías obtener el valor de este campo en el controlador con `params[:query]`.

Cuando nombra los inputs, Rails utiliza ciertas convenciones que hacen posible enviar parámetros con valores no escalares tales como arrays o hashes, que también serán accesibles en `params`. Puede leer más sobre ellos en el capítulo 7 de esta guía. Para obtener detalles sobre el uso preciso de estos ayudantes, consulte la documentación de la API.

#### 1.3.1 Casillas de verificación

Las casillas de verificación son controles de formulario que proporcionan al usuario un conjunto de opciones que pueden activar o desactivar:

```ruby
<%= check_box_tag(:pet_dog) %>
<%= label_tag(:pet_dog, "I own a dog") %>
<%= check_box_tag(:pet_cat) %>
<%= label_tag(:pet_cat, "I own a cat") %>
```

Esto genera lo siguiente:

```ruby
<input id="pet_dog" name="pet_dog" type="checkbox" value="1" />
<label for="pet_dog">I own a dog</label>
<input id="pet_cat" name="pet_cat" type="checkbox" value="1" />
<label for="pet_cat">I own a cat</label>
```

El primer parámetro para `check_box_tag`, por supuesto, es el nombre del `input`. El segundo parámetro, naturalmente, es el valor del `input`. Este valor se incluirá en los datos del formulario \(y estará presente en los parámetros\) cuando se marque la casilla de verificación.

#### 1.3.2 Radio buttons

Los radio buttons, aunque son similares a las casillas de verificación, son controles que especifican un conjunto de opciones en las que son mutuamente excluyentes \(es decir, el usuario sólo puede elegir uno\):

```ruby
<%= radio_button_tag(:age, "child") %>
<%= label_tag(:age_child, "I am younger than 21") %>
<%= radio_button_tag(:age, "adult") %>
<%= label_tag(:age_adult, "I'm over 21") %>
```

Salida:

```ruby
<input id="age_child" name="age" type="radio" value="child" />
<label for="age_child">I am younger than 21</label>
<input id="age_adult" name="age" type="radio" value="adult" />
<label for="age_adult">I'm over 21</label>
```

Al igual que con `check_box_tag`, el segundo parámetro de `radio_button_tag` es el valor del input. Debido a que estos dos radio buttons comparten el mismo nombre \(`age`\), el usuario sólo podrá seleccionar uno de ellos y `params[:age]` contendrá "child" o "adult".

> Utilice siempre las etiquetas de checkbox o radio buttons. Asocian el texto a una opción específica y, al expandir la región seleccionable, facilitan que los usuarios hagan clic en las entradas.





### 1.4 Otros ayudantes de interés

Otros controles de formulario que vale la pena mencionar son campos de texto, campos de contraseña, campos ocultos, campos de búsqueda, campos de teléfono, campos de fecha, campos de tiempo, campos de color, campos de fecha y hora, campos de mes, campos de semana, campos de URL, campos de correo electrónico, campos de números y campos de rango:

```ruby
<%= text_area_tag(:message, "Hi, nice site", size: "24x6") %>
<%= password_field_tag(:password) %>
<%= hidden_field_tag(:parent_id, "5") %>
<%= search_field(:user, :name) %>
<%= telephone_field(:user, :phone) %>
<%= date_field(:user, :born_on) %>
<%= datetime_local_field(:user, :graduation_day) %>
<%= month_field(:user, :birthday_month) %>
<%= week_field(:user, :birthday_week) %>
<%= url_field(:user, :homepage) %>
<%= email_field(:user, :address) %>
<%= color_field(:user, :favorite_color) %>
<%= time_field(:task, :started_at) %>
<%= number_field(:product, :price, in: 1.0..20.0, step: 0.5) %>
<%= range_field(:product, :discount, in: 1..100) %>
```

Salida

```ruby
<textarea id="message" name="message" cols="24" rows="6">Hi, nice site</textarea>
<input id="password" name="password" type="password" />
<input id="parent_id" name="parent_id" type="hidden" value="5" />
<input id="user_name" name="user[name]" type="search" />
<input id="user_phone" name="user[phone]" type="tel" />
<input id="user_born_on" name="user[born_on]" type="date" />
<input id="user_graduation_day" name="user[graduation_day]" type="datetime-local" />
<input id="user_birthday_month" name="user[birthday_month]" type="month" />
<input id="user_birthday_week" name="user[birthday_week]" type="week" />
<input id="user_homepage" name="user[homepage]" type="url" />
<input id="user_address" name="user[address]" type="email" />
<input id="user_favorite_color" name="user[favorite_color]" type="color" value="#000000" />
<input id="task_started_at" name="task[started_at]" type="time" />
<input id="product_price" max="20.0" min="1.0" name="product[price]" step="0.5" type="number" />
<input id="product_discount" max="100" min="1" name="product[discount]" type="range" />
```

Las entradas ocultas no se muestran al usuario, sino que en su lugar contienen datos como cualquier entrada textual. Los valores dentro de ellos se pueden cambiar con JavaScript.

> Las entradas de búsqueda, teléfono, fecha, hora, color, fecha, fecha, hora, mes, semana, URL, correo electrónico, número y rango son controles HTML5. Si requiere que su aplicación tenga una experiencia consistente en navegadores antiguos, necesitará un `polyfill` HTML5 \(proporcionado por CSS y / o JavaScript\). Definitivamente no hay escasez de soluciones para esto, aunque una herramienta popular en este momento es `Modernizr`, que proporciona una forma sencilla de agregar funcionalidad basada en la presencia de características HTML5 detectadas.

> Si utiliza campos de entrada de contraseña \(para cualquier propósito\), puede configurar su aplicación para evitar que se registren esos parámetros. Puede obtener más información al respecto en la Guía de seguridad.



















