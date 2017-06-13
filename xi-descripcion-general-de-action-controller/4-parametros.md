# 4- Parámetros

Es probable que desee acceder a los datos enviados por el usuario u otros parámetros en las acciones del controlador. Hay dos tipos de parámetros posibles en una aplicación web. Los primeros son los parámetros que se envían como parte de la URL, llamados parámetros de la cadena de consulta. La cadena de consulta es todo lo que hay después de "`?`" En la URL. El segundo tipo de parámetro se denomina normalmente datos `POST`. Esta información normalmente proviene de un formulario HTML que ha sido llenado por el usuario. Se llama datos `POST` porque sólo se puede enviar como parte de una solicitud HTTP POST. Rails no hace ninguna distinción entre los parámetros de cadena de consulta y los parámetros de POST, y ambos están disponibles en el hash params en su controlador:

```ruby
class ClientsController < ApplicationController
  # Esta acción utiliza parámetros de cadena de consulta porque se ejecuta
  # por una solicitud HTTP GET, pero esto no hace ninguna diferencia
  # a la forma en que se accede a los parámetros. La URL para
  # esta acción se vería así en la URL con la lista activada
  # clients: /clients?status=activated
  def index
    if params[:status] == "activated"
      @clients = Client.activated
    else
      @clients = Client.inactivated
    end
  end

  # Esta acción utiliza parámetros POST. Lo más probable es que vengan
  # de un formulario HTML que el usuario ha enviado. La URL para
  # esta solicitud RESTful será "/clients", y los datos serán
  # enviados como parte del cuerpo de la solicitud.  
  def create
    @client = Client.new(params[:client])
    if @client.save
      redirect_to @client
    else
      render "new"
    end
  end
end
```

### 4.1 Parámetros de hash y de array

El hash de parámetros no está limitado a claves y valores unidimensionales. Puede contener arrays anidados y hashes. Para enviar un array de valores, agregue un par de corchetes vacíos "`[]`" al nombre de la clave:

```ruby
GET /clients?ids[]=1&ids[]=2&ids[]=3
```

> La URL real en este ejemplo se codificará como "`/clients?ids%5b%5d=1&ids%5b%5d=2&ids%5b%5d=3`" ya que los caracteres "`[`" y "`]`" no están permitidos en las URL. La mayoría de las veces no tienes que preocuparte por esto porque el navegador lo codificará por ti, y Rails lo descodificará automáticamente, pero si alguna vez te encuentras teniendo que enviar esas solicitudes al servidor manualmente debes tener esto en cuenta .

El valor de `params[:ids]` será ahora`["1", "2", "3"]`. Tenga en cuenta que los valores de los parámetros son siempre cadenas; Rails no intenta adivinar o lanzar el `type`.

> Valores como `[nil]` o`[nil, nil, ...]` en params se sustituyen por `[]`por motivos de seguridad de forma predeterminada. Consulte la Guía de seguridad para obtener más información.

Para enviar un hash, debe incluir el nombre de la clave dentro de los corchetes:

```ruby
<form accept-charset="UTF-8" action="/clients" method="post">
  <input type="text" name="client[name]" value="Acme" />
  <input type="text" name="client[phone]" value="12345" />
  <input type="text" name="client[address][postcode]" value="12345" />
  <input type="text" name="client[address][city]" value="Carrot City" />
</form>
```

Cuando se envíe este formulario, el valor de `params[:client]` será `{ "name" => "Acme", "phone" => "12345", "address" => { "postcode" => "12345", "city" => "Carrot City" } }`. Tenga en cuenta el hash anidado en `params[:client][: address]`.

El objeto `params` actúa como un Hash, pero le permite usar símbolos y cadenas de forma intercambiable como claves.

### 4.2 Parámetros JSON

Si está escribiendo una API, es posible que se encuentre más cómodo aceptando parámetros en formato `JSON`. Si el encabezado "`Content-Type`" de su solicitud está configurado como "`application/json`", Rails cargará automáticamente sus parámetros en el hash de parámetros, al que puede acceder como lo haría normalmente.

Por ejemplo, si envía este contenido de `JSON`:

```ruby
{ "company": { "name": "acme", "address": "123 Carrot Street" } }
```

Su controlador recibirá `params[:company]` como `{"name" => "acme", "address" => "123 Carrot Street"}`.

Además, si ha activado `config.wrap_parameters` en su inicializador o ha llamado `wrap_parameters` en su controlador, puede omitir con seguridad el elemento raíz en el parámetro `JSON`. En este caso, los parámetros serán clonados y envueltos con una clave elegida basada en el nombre de su controlador. Por lo tanto, la petición `JSON` anterior puede escribirse como:

```ruby
{ "name": "acme", "address": "123 Carrot Street" }
```

Y, asumiendo que usted está enviando los datos a `CompaniesController`, entonces sería envuelto dentro de la llave `:company` así:

```ruby
{ name: "acme", address: "123 Carrot Street", company: { name: "acme", address: "123 Carrot Street" } }
```

Puede personalizar el nombre de la clave o los parámetros específicos que desea ajustar consultando la documentación de la API

> El soporte para parsear parámetros `XML` se ha extraído en una gema denominada `actionpack-xml_parser`.





### 4.3 Parámetros de enrutamiento

El hash params siempre contendrá las claves `:controller` y `:action`, pero debería usar los métodos `controller_name` y `action_name` para acceder a estos valores. Otros parámetros definidos por el enrutamiento, tales como `:id`, también estarán disponibles. Por ejemplo, considere una lista de clientes donde la lista puede mostrar clientes activos o inactivos. Podemos agregar una ruta que captura el parámetro `:status` en una URL "bonita":

```ruby
get '/clients/:status' => 'clients#index', foo: 'bar'
```

En este caso, cuando un usuario abre la URL `/clients/active`, `params[:status]` se establecerá en "`active`". Cuando se utiliza esta ruta, `params[:foo]` también se establecerá en "`bar`", como si se pasara en la cadena de consulta. Su controlador también recibirá `params[:action]` como "`index`" y `params[:controller]` como "`clients`".





### 4.4 default\_url\_options

Puede definir parámetros globales predeterminados para la generación de las URLs definiendo un método denominado `default_url_options` en su controlador. Tal método debe devolver un hash con los valores predeterminados deseados, cuyas keys deben ser símbolos:

```ruby
class ApplicationController < ActionController::Base
  def default_url_options
    { locale: I18n.locale }
  end
end
```

Estas opciones se utilizarán como punto de partida al generar las URLs, por lo que es posible que se anulen por las opciones que se pasan a `url_for` en las llamadas.

Si define `default_url_options` en `ApplicationController`, como en el ejemplo anterior, estos valores predeterminados se utilizarán para toda la generación de todas las URLs. El método también puede definirse en un controlador específico, en cuyo caso sólo afecta a las URLs allí generadas.

En una solicitud dada, el método no es realmente llamado para cada URL generada; Por razones de rendimiento, el hash devuelto se almacena en caché, y hay como máximo una invocación por solicitud.





### 4.5 Strong Parameters

Con strong parameter, los parámetros de Action Controller están prohibidos para ser utilizados en las asignaciones masivas del Active Model hasta que hayan sido incluidos en la lista blanca \(white listed\). Esto significa que usted tendrá que tomar una decisión consciente sobre qué atributos permite la actualización masiva. Esta es una mejor práctica de seguridad, para ayudar a evitar accidentes, al permitir a los usuarios actualizar los atributos de modelos sensibles.

Además, los parámetros se pueden marcar como se requiera y fluirán a través de un flujo de raise/rescue predefinido que dará lugar a que se devuelva una 400 Bad Request si no se pasan todos los parámetros requeridos.

```ruby
class PeopleController < ActionController::Base
  # Esto disparará una excepción ActiveModel::ForbiddenAttributesError
  # porque está utilizando una asignación masiva sin un permiso explícito
  # para este paso.
  def create
    Person.create(params[:person])
  end

  # Esto pasará sin problemas, siempre y cuando hay una key person
  # en los parámetros, de lo contrario dispara una excepción
  # ActionController::ParameterMissing, que obtendrá una captura
  # de ActionController::Base y lo convierte en un error 400 Bad
  # Request.
  def update
    person = current_account.people.find(params[:id])
    person.update!(person_params)
    redirect_to person
  end

  private
    # Utilizando un método privado para encapsular los parámetros permitidos
    # es sólo un buen patrón ya que podrá reutilizar el mismo
    # permit del whitelist en create y update. Además, puede especializarse
    # este método con la verificación por usuario de los atributos permitidos.
    def person_params
      params.require(:person).permit(:name, :age)
    end
end
```

#### 4.5.1 Valores escalares permitidos

Dado

