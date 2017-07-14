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

La variable de interpolación`:count` tiene un papel especial en que se interpola a la traducción y se utiliza para escoger una pluralización a partir de las traducciones según las reglas de pluralización definidas por CLDR:

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

Entonces `User.model_name.human` devolverá "Dude" y `User.human_attribute_name("login")` devolverá "Handle".

También puede establecer una forma plural para los nombres de los modelos, agregandolos como sigue:

```ruby
en:
  activerecord:
    models:
      user:
        one: Dude
        other: Dudes
```

Entonces `User.model_name.human(count: 2)` devolverá "Dudes". Con `count: 1` o sin params devolverá "Dude".

En caso de que necesite acceder a atributos anidados dentro de un modelo determinado, debe anidarlos bajo `model/attribute` en el nivel de modelo de su archivo de traducción:

```ruby
en:
  activerecord:
    attributes:
      user/gender:
        female: "Female"
        male: "Male"
```

Entonces `User.human_attribute_name("gender.female")` devolverá "Female".

> Si está utilizando una clase que incluye ActiveModel y no hereda de `ActiveRecord::Base`, reemplace activerecord con activemodel en las rutas de clave anteriores.

#### 4.5.1 Scope de los mensajes de error

Los mensajes de error de validación de Active record también se pueden traducirse fácilmente. Active Record le ofrece un par de espacios de nombres donde puede colocar sus traducciones de mensajes con el fin de proporcionar diferentes mensajes y traducción para ciertos modelos, atributos y / o validaciones. También tiene en cuenta transparentemente la herencia de una sola tabla.

Esto le da medios muy poderosos para ajustar con flexibilidad sus mensajes a las necesidades de su aplicación.

Considere un modelo de usuario con una validación para el atributo de nombre como este:

```ruby
class User < ApplicationRecord
  validates :name, presence: true
end
```

La clave para el mensaje de error en este caso es `:blank`. Active Record buscará esta clave en los espacios de nombres:

```ruby
activerecord.errors.models.[model_name].attributes.[attribute_name]
activerecord.errors.models.[model_name]
activerecord.errors.messages
errors.attributes.[attribute_name]
errors.messages
```

Así, en nuestro ejemplo probará las siguientes claves en este orden y devolverá el primer resultado:

```ruby
activerecord.errors.models.user.attributes.name.blank
activerecord.errors.models.user.blank
activerecord.errors.messages.blank
errors.attributes.name.blank
errors.messages.blank
```

Cuando sus modelos también están usando la herencia entonces los mensajes se buscan en la cadena de herencia.

Por ejemplo, es posible que tenga un modelo Admin heredado de Usuario:

```ruby
class Admin < User
  validates :name, presence: true
end
```

Entonces Active Record buscará mensajes en este orden:

```ruby
activerecord.errors.models.admin.attributes.name.blank
activerecord.errors.models.admin.blank
activerecord.errors.models.user.attributes.name.blank
activerecord.errors.models.user.blank
activerecord.errors.messages.blank
errors.attributes.name.blank
errors.messages.blank
```

De esta manera, puede proporcionar traducciones especiales para varios mensajes de error en diferentes puntos de la cadena de herencia de los modelos y en los atributos, modelos o ámbitos predeterminados.

#### 4.5.2 Interpolación de mensajes de error

El nombre del modelo traducido, el nombre del atributo traducido y el valor siempre están disponibles para la interpolación como modelo, atributo y valor, respectivamente.

Por ejemplo, en lugar del mensaje de error predeterminado "cannot be blank", puede utilizar el nombre de atributo como este: "Please fill in your %{attribute}".

Cuando esté disponible, puede utilizarse para la pluralización si está presente:

| validation | with option | message | interpolation |
| :--- | :--- | :--- | :--- |
| confirmation | - | :confirmation | attribute |
| acceptance | - | :accepted | - |
| presence | - | :blank | - |
| absence | - | :present | - |
| length | :within, :in | :too\_short | count |
| length | :within, :in | :too\_long | count |
| length | :is | :wrong\_length | count |
| length | :minimum | :too\_short | count |
| length | :maximum | :too\_long | count |
| uniqueness | - | :taken | - |
| format | - | :invalid | - |
| inclusion | - | :inclusion | - |
| exclusion | - | :exclusion | - |
| associated | - | :invalid | - |
| non-optional association | - | :required | - |
| numericality | - | :not\_a\_number | - |
| numericality | :greater\_than | :greater\_than | count |
| numericality | :greater\_than\_or\_equal\_to | :greater\_than\_or\_equal\_to | count |
| numericality | :equal\_to | :equal\_to | count |
| numericality | :less\_than | :less\_than | count |
| numericality | :less\_than\_or\_equal\_to | :less\_than\_or\_equal\_to | count |
| numericality | :other\_than | :other\_than | count |
| numericality | :only\_integer | :not\_an\_integer | - |
| numericality | :odd | :odd | - |
| numericality | :even | :even | - |

#### 4.5.3 Traducciones para Helper `error_messages_for` de Active Record

Rails incluye las siguientes traducciones:

```ruby
en:
  activerecord:
    errors:
      template:
        header:
          one:   "1 error prohibited this %{model} from being saved"
          other: "%{count} errors prohibited this %{model} from being saved"
        body:    "There were problems with the following fields:"
```

Para usar este ayudante, debe instalar la gema `DynamicForm` añadiendo esta línea a su Gemfile: `gem 'dynamic_form'`.

#### 4.6 Traducciones del Subject de Email de Action Mailer.

Si no pasa un asunto al método de correo, Action Mailer intentará encontrarlo en sus traducciones. La búsqueda realizada utilizará el patrón `<mailer_scope>`. `<action_name> .subject` para construir la clave.

```ruby
# user_mailer.rb
class UserMailer < ActionMailer::Base
  def welcome(user)
    #...
  end
end
```

```ruby
en:
  user_mailer:
    welcome:
      subject: "Welcome to Rails Guides!"
```

Para enviar parámetros a la interpolación, use el método `default_i18n_subject` en el mailer.

```ruby
# user_mailer.rb
class UserMailer < ActionMailer::Base
  def welcome(user)
    mail(to: user.email, subject: default_i18n_subject(user: user.name))
  end
end
```

```ruby
en:
  user_mailer:
    welcome:
      subject: "%{user}, welcome to Rails Guides!"
```

#### 4.7 Descripción general de otros métodos integrados que proporcionan compatibilidad con I18n

Rails utiliza cadenas fijas y otras localizaciones, como cadenas de formato y otra información de formato en un par de ayudantes. He aquí un breve resumen.

##### 4.7.1 Métodos de ayuda de Action View

* `distance_of_time_in_words` traduce y pluraliza su resultado e interpola el número de segundos, minutos, horas, etc. Vea las traducciones de `datetime.distance_in_words`.
* `datetime_select` y `select_month` utilizan nombres de mes traducidos para rellenar la etiqueta de selección resultante. Vea `date.month_names` para traducciones. `datetime_select` también busca la opción de orden desde `date.order` \(a menos que pase la opción explícitamente\). Todos los ayudantes de selección de fechas traducen el mensaje utilizando las traducciones en el ámbito `datetime.prompts` si corresponde.
* Los helpers `number_to_currency`, `number_with_precision`, `number_to_percentage`, `number_with_delimiter` y `number_to_human_size` utilizan los ajustes de formato numérico situados en el ámbito numérico.

##### 4.7.2 Métodos del Active Model

* `model_name.human` y `human_attribute_name` utilizan traducciones para nombres de modelos y nombres de atributos si están disponibles en el ámbito `activerecord.models`. También admiten traducciones de nombres de clases heredados \(por ejemplo, para su uso con STI\) como se explicó anteriormente en "Scope de mensaje de error".
* `ActiveModel::Errors#generate_message` \(que es usado por las validaciones de Active Model pero también se puede usar manualmente\) usa `nombre_modelo.human` y `human_attribute_name` \(ver arriba\). También traduce el mensaje de error y admite traducciones para nombres de clases heredados como se explicó anteriormente en "Scope de mensaje de error".
* `ActiveModel::Errors#full_messages` anula el nombre del atributo al mensaje de error usando un separador que se buscará desde `errors.format` \(y que por defecto es "`%{attribute} %{message}`"\).

##### 4.7.3 Métodos de Active Support

`Array#to_sentence` utiliza la configuración de formato tal como se da en el ámbito `support.array`.





