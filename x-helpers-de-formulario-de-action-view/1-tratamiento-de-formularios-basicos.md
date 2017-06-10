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















