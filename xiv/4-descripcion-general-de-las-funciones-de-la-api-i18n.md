# 4- Descripción general de las funciones de la API I18n

Debe tener una buena comprensión ahora para utilizar la biblioteca `i18n`  y saber cómo internacionalizar una aplicación de Rails básica. En los capítulos siguientes, cubriremos sus características con más profundidad.

Estos capítulos mostrarán ejemplos utilizando tanto el método I18n.translate como el método auxiliar de traducción de la vista \(observando la característica adicional proporcionada por el método de ayuda de la vista\).

Cubriremos características como estas:

* Buscar traducciones
* Interpolando datos en traducciones
* Traducciones pluralizadas
* Usando traducciones seguras de HTML \(solo ver el método auxiliar\)
* Localización de fechas, números, moneda, etc.

### 4.1 Buscar traducciones

#### 4.1.1 Búsqueda básica, scopes y claves anidadas

Las traducciones son buscadas por claves que pueden ser Símbolos o Strings, por lo que estas llamadas son equivalentes:

```ruby
I18n.t :message
I18n.t 'message'
```

El método de traducción también tiene una opción `:scope` que puede contener una o más claves adicionales que se utilizarán para especificar un "espacio de nombres" o el alcance para una clave de la traducción:

```ruby
I18n.t :record_invalid, scope: [:activerecord, :errors, :messages]
```

Esto busca el mensaje: `record_invalid` en los mensajes de error de Active Record.

Además, tanto la clave como los ámbitos pueden especificarse como claves separadas por puntos como en:

```ruby
I18n.translate "activerecord.errors.messages.record_invalid"
```

Así, las siguientes llamadas son equivalentes:

```ruby
I18n.t 'activerecord.errors.messages.record_invalid'
I18n.t 'errors.messages.record_invalid', scope: :activerecord
I18n.t :record_invalid, scope: 'activerecord.errors.messages'
I18n.t :record_invalid, scope: [:activerecord, :errors, :messages]
```

4.1.2 Default

Cuando se da una opción `:default`, su valor será devuelto si falta la traducción:

```ruby
I18n.t :missing, default: 'Not here'
# => 'Not here'
```

Si el valor predeterminado es un símbolo, se utilizará como clave y se traducirá. Uno puede proporcionar varios valores como predeterminados. Se devolverá el primero que resulte en un valor.

Por ejemplo, el siguiente trata de traducir la clave `:missing` y luego la clave `:also_missing`. Como ambos no producen un resultado, la cadena "Not here" será devuelta:

```ruby
I18n.t :missing, default: [:also_missing, 'Not here']
# => 'Not here'
```

##### 4.1.3 Búsqueda en masa \(bulk\) y espacio de nombres

Para buscar múltiples traducciones a la vez, se puede pasar una matriz de claves:

```ruby
I18n.t [:odd, :even], scope: 'errors.messages'
# => ["must be odd", "must be even"]
```

Además, una clave puede traducirse a un hash \(potencialmente anidado\) de traducciones agrupadas. Por ejemplo, uno puede recibir todos los mensajes de error Active Record como un hash con:

```ruby
I18n.t 'activerecord.errors.messages'
# => {:inclusion=>"is not included in the list", :exclusion=> ... }
```

##### 4.1.4 Búsqueda "perezosa"

Rails implementa una forma conveniente de buscar la ubicación dentro de las vistas. Cuando tenga el siguiente diccionario:

```ruby
es:
  books:
    index:
      title: "Título"
```

Puede buscar el valor `books.index.title` dentro de la aplicación `/views/books/index.html.erb` en una plantilla como esta \(tenga en cuenta el punto\):

```ruby
<%= t '.title' %>
```

El alcance de la traducción automática por parcial sólo está disponible en el método de ayuda de la vista de traducción.

La búsqueda "perezosa" también se puede usar en los controladores:

```ruby
en:
  books:
    create:
      success: Book created!
```

Esto es útil para configurar mensajes de flash como por ejemplo:

```ruby
class BooksController < ApplicationController
  def create
    # ...
    redirect_to books_url, notice: t('.success')
  end
end
```

### 4.2 Pluralización

En inglés hay sólo una forma singular y una plural para una cadena dada, por ejemplo. "1 mensaje" y "2 mensajes". Otros idiomas \(árabe, japonés, ruso y muchos más\) tienen diferentes gramáticas que tienen formas plurales adicionales o menos. Por lo tanto, la I18n API proporciona una funcionalidad flexible de pluralización.

La variable de interpolación` :count` tiene un papel especial en que se interpola a la traducción y se utiliza para escoger una pluralización a partir de las traducciones según las reglas de pluralización definidas por CLDR:

```ruby
I18n.backend.store_translations :en, inbox: {
  zero: 'no messages', # optional
  one: 'one message',
  other: '%{count} messages'
}
I18n.translate :inbox, count: 2
# => '2 messages'
 
I18n.translate :inbox, count: 1
# => 'one message'
 
I18n.translate :inbox, count: 0
# => 'no messages'
```

El algoritmo para pluralizaciones para `:en` es tan simple como:

```ruby
lookup_key = :zero if count == 0 && entry.has_key?(:zero)
lookup_key ||= count == 1 ? :one : :other
entry[lookup_key]
```

La traducción denotada como `:one` se considera singular, y el otro se usa como plural. Si el recuento es cero y una entrada cero está presente, entonces se utilizará `:other` en su lugar.

Si la búsqueda de la clave no devuelve un hash adecuado para la pluralización, se genera una excepción `I18n::InvalidPluralizationData`.

```ruby
I18n.locale = :de
I18n.t :foo
I18n.l Time.now
```

Pasando explícitamente una configuración regional:

```ruby
I18n.t :foo, locale: :de
I18n.l Time.now, locale: :de
```

El `I18n.locale` tiene por defecto `I18n.default_locale` cuyo valor predeterminado es `:en`. La configuración regional predeterminada se puede establecer de la siguiente manera:

```ruby
I18n.default_locale = :de
```

### 4.4 Uso de las traducciones HTML seguras

Las claves con un sufijo '`_html`' y las claves denominadas 'html' están marcadas como seguras para HTML. Cuando los uses en las vistas, el código HTML no se escapará.

```ruby
# config/locales/en.yml
en:
  welcome: <b>welcome!</b>
  hello_html: <b>hello!</b>
  title:
    html: <b>title!</b>
```

```ruby
# app/views/home/index.html.erb
<div><%= t('welcome') %></div>
<div><%= raw t('welcome') %></div>
<div><%= t('hello_html') %></div>
<div><%= t('title.html') %></div>
```

Sin embargo, la interpolación se escapa según sea necesario. Por ejemplo, dado:

```ruby
en:
  welcome_html: "<b>Welcome %{username}!</b>"
```

Puede pasar con seguridad el nombre de usuario según lo establecido por el usuario:

```ruby
<%# This is safe, it is going to be escaped if needed. %>
<%= t('welcome_html', username: @current_user.username) %>
```

Las cadenas seguras por otro lado se interpolan literalmente.

> La conversión automática al texto de traducción HTML seguro sólo está disponible desde el método de ayuda de traducción de las vistas.

![](/assets/Captura de pantalla 2017-07-14 a las 3.12.28 p.m..png)

### 4.5 Traducciones de Active Record Models

Puede utilizar los métodos `Model.model_name.human` y `Model.human_attribute_name(attribute)` para buscar traducciones transparentes para los nombres de modelo y atributo.

Por ejemplo, al agregar las siguientes traducciones:

```ruby
en:
  activerecord:
    models:
      user: Dude
    attributes:
      user:
        login: "Handle"
      # will translate User attribute "login" as "Handle"
```









