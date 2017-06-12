# 7- Entendiendo las convenciones de nombramiento de parámetros

Como se ha visto en las secciones anteriores, los valores de los formularios pueden estar en el nivel superior del hash `params` o anidados en otro hash. Por ejemplo, en una acción `create` estándar para un modelo `Person`, `params[:person]` normalmente sería un hash de todos los atributos para la persona a crear. El hash de parámetros también puede contener arrays, arrays de hashes y así sucesivamente.

Fundamentalmente los formularios `HTML` no conocen ningún tipo de datos estructurados, todo lo que generan son pares nombre-valor, donde los pares son simplemente cadenas simples. Los arrays y hashes que ve en su aplicación son el resultado de algunas convenciones de nombre de parámetros que Rails utiliza.





### 7.1 Estructuras Básicas

Las dos estructuras básicas son arrays y hashes. Los hash reflejan la sintaxis utilizada para acceder al valor en los parámetros. Por ejemplo, si un formulario contiene:

```ruby
<input id="person_name" name="person[name]" type="text" value="Henry"/>
```

El hash params contendrá:

```ruby
{'person' => {'name' => 'Henry'}}
```

Y `params[:person][:name]` recuperará el valor enviado en el controlador.

Las hashes pueden anidarse en tantos niveles como se requiera, por ejemplo:

```ruby
<input id="person_address_city" name="person[address][city]" type="text" value="New York"/>
```

Resultará en el hash params:

```ruby
{'person' => {'address' => {'city' => 'New York'}}}
```

Normalmente, Rails ignora los nombres de parámetros duplicados. Si el nombre del parámetro contiene un conjunto vacío de corchetes`[]`, entonces se acumulan en un array. Si desea que los usuarios puedan ingresar varios números de teléfono, puede colocarlo en la forma siguiente:

```ruby
<input name="person[phone_number][]" type="text"/>
<input name="person[phone_number][]" type="text"/>
<input name="person[phone_number][]" type="text"/>
```

Esto resultaría en `params[:person][:phone_number]` siendo un array que contiene los números de teléfono introducidos.





### 7.2 Combinándolos

Podemos mezclar y combinar estos dos conceptos. Un elemento de un hash podría ser un array como en el ejemplo anterior, o puede tener un array de hashes. Por ejemplo, un formulario puede permitirle crear cualquier número de direcciones repitiendo el siguiente fragmento de formulario

```ruby
<input name="addresses[][line1]" type="text"/>
<input name="addresses[][line2]" type="text"/>
<input name="addresses[][city]" type="text"/>
```

Esto resultaría en `params[:addresses]` siendo un array de hashes con las claves `line1`, `line2` y `city`. Rails decide comenzar a acumular valores en un nuevo hash cada vez que encuentre un input `name` que ya existe en el hash actual.

Hay una restricción, sin embargo, mientras que los hashes pueden anidarse arbitrariamente, solo se permite un nivel de "arrayness". Los arrays pueden usualmente ser reemplazadas por hashes; Por ejemplo, en lugar de tener un array de objetos de un modelo, se puede tener un hash de objetos de un modelo indexados por su `id`, un array de indices o algún otro parámetro.

> Los parámetros de un array no funcionan bien con el helper `check_box`. De acuerdo con las casillas de verificación desmarcadas de la especificación HTML no se envía ningún valor. Sin embargo, a menudo es conveniente que una casilla de verificación siempre envíe un valor. El helper `check_box` simula esto creando un input auxiliar oculto con el mismo name. Si la casilla de verificación está desmarcada, sólo se enviará el input oculto y si se marca, ambos se enviarán, pero el valor enviado por la casilla de verificación tendrá prioridad. Al trabajar con parámetros de un array, este envío duplicado confundirá a Rails ya que los input names duplicados son la forma en que decide cuándo se debe iniciar un nuevo elemento array. Es preferible usar `check_box_tag` o usar hashes en lugar de arrays.





### 7.3 Uso de los helpers de formularios

Las secciones anteriores no utilizaron los helpers de los formularios de Rails en absoluto. Mientras que usted puede elaborar los input names usted mismo y pasarlos directamente a los helpers como `text_field_tag` Rails también proporciona un soporte de nivel superior. Las dos herramientas a su disposición aquí son el parámetro `name` para `form_for` y `fields_for` y la opción `:index` que los helpers toman.

```ruby
<%= form_for @person do |person_form| %>
  <%= person_form.text_field :name %>
  <% @person.addresses.each do |address| %>
    <%= person_form.fields_for address, index: address.id do |address_form|%>
      <%= address_form.text_field :city %>
    <% end %>
  <% end %>
<% end %>
```

Suponiendo que la persona tenía dos direcciones, con `ids` 23 y 45 esto crearía una salida similar a esto:

```ruby
<form accept-charset="UTF-8" action="/people/1" class="edit_person" id="edit_person_1" method="post">
  <input id="person_name" name="person[name]" type="text" />
  <input id="person_address_23_city" name="person[address][23][city]" type="text" />
  <input id="person_address_45_city" name="person[address][45][city]" type="text" />
</form>
```

Esto resultará en un hash params que se parece a:

```ruby
{'person' => {'name' => 'Bob', 'address' => {'23' => {'city' => 'Paris'}, '45' => {'city' => 'London'}}}}
```

Rails sabe que todas estas entradas deben ser parte del hash de `Person` porque has llamado `fields_for` en el primer constructor de formularios. Al especificar una opción `:index`, le está diciendo a Rails que en lugar de nombrar los inputs `person[address][city]`, debe insertar ese index rodeado de `[]` entre la dirección y la ciudad. Esto es a menudo útil ya que es fácil localizar qué registro de dirección debe ser modificado. Puede pasar números con alguna otra significación, cadenas o incluso `nil` \(lo que resultará en un parámetro array que se está creando\).

```ruby
<%= fields_for 'person[address][primary]', address, index: address do |address_form| %>
  <%= address_form.text_field :city %>
<% end %>
```

Creará inputs como:

```ruby
<input id="person_address_primary_1_city" name="person[address][primary][1][city]" type="text" value="bologna" />
```

Como regla general, el nombre del input final es la concatenación del name dado a fields\_for / `form_for`, el valor del index y el nombre del atributo. También puede pasar una opción index directamente a helpers como `text_field`, pero suele ser menos repetitivo especificar esto a nivel del constructor de formularios en lugar de en controles de input individuales.

```ruby
<%= fields_for 'person[address][primary][]', address do |address_form| %>
  <%= address_form.text_field :city %>
<% end %>
```

Produce exactamente la misma salida que el ejemplo anterior.

