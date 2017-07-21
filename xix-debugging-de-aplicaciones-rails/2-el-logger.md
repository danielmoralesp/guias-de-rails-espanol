# 2- El Logger

También puede ser útil guardar información en archivos log en tiempo de ejecución. Rails mantiene un archivo log separado para cada entorno de tiempo de ejecución.

### 2.1 ¿Qué es el logger?

Rails hace uso de la clase `ActiveSupport::Logger` para escribir información de los logs. Otros logger, como `Log4r`, también pueden ser sustituidos.

Puede especificar un logger alternativo en `config/application.rb` o en cualquier otro archivo de entorno, por ejemplo:

```ruby
config.logger = Logger.new(STDOUT)
config.logger = Log4r::Logger.new("Application Log")
```

O bien, en la sección `initializer`, agregue cualquiera de las siguientes opciones

```ruby
Rails.logger = Logger.new(STDOUT)
Rails.logger = Log4r::Logger.new("Application Log")
```

De forma predeterminada, cada registro se crea en `Rails.root/log/` y el archivo de logs lleva el nombre del entorno en el que se ejecuta la aplicación.

### 2.2 Niveles de logs

Cuando algo se registra, se imprime en el logs correspondiente si el nivel de log del mensaje es igual o superior al nivel de log configurado. Si desea conocer el nivel de log actual, puede llamar al método `Rails.logger.level`.

Los niveles de log disponibles son: `:debug`, `:info`, :`warn`, `:error,` `:fatal`, y `:unknown`, correspondiente a los números de nivel de log de 0 a 5, respectivamente. Para cambiar el nivel de log predeterminado, use:

```ruby
config.log_level = :warn # In any environment initializer, or
Rails.logger.level = 0 # at any time
```

Esto es útil cuando desea iniciar sesión bajo develpment o staging sin inundar su log de producción con información innecesaria.

> El nivel de log predeterminado de Rails es depurado en todos los entornos.

### 2.3 Envío de mensajes

Para escribir en el log actual, utilice el método `logger.(debug|info|warn|error|fatal)` desde un controlador, modelo o mailer:

```ruby
logger.debug "Person attributes hash: #{@person.attributes.inspect}"
logger.info "Processing the request..."
logger.fatal "Terminating application, raised unrecoverable error!!!"
```

Aquí está un ejemplo de un método instrumentado con logging adicional:

```ruby
class ArticlesController < ApplicationController
  # ...
 
  def create
    @article = Article.new(params[:article])
    logger.debug "New article: #{@article.attributes.inspect}"
    logger.debug "Article should be valid: #{@article.valid?}"
 
    if @article.save
      flash[:notice] =  'Article was successfully created.'
      logger.debug "The article was saved and now the user is going to be redirected..."
      redirect_to(@article)
    else
      render action: "new"
    end
  end
 
  # ...
end
```

A continuación se muestra un ejemplo del log generado cuando se ejecuta esta acción del controlador:

```bash
Processing ArticlesController#create (for 127.0.0.1 at 2008-09-08 11:52:54) [POST]
  Session ID: BAh7BzoMY3NyZl9pZCIlMDY5MWU1M2I1ZDRjODBlMzkyMWI1OTg2NWQyNzViZjYiCmZsYXNoSUM6J0FjdGl
vbkNvbnRyb2xsZXI6OkZsYXNoOjpGbGFzaEhhc2h7AAY6CkB1c2VkewA=--b18cd92fba90eacf8137e5f6b3b06c4d724596a4
  Parameters: {"commit"=>"Create", "article"=>{"title"=>"Debugging Rails",
 "body"=>"I'm learning how to print in logs!!!", "published"=>"0"},
 "authenticity_token"=>"2059c1286e93402e389127b1153204e0d1e275dd", "action"=>"create", "controller"=>"articles"}
New article: {"updated_at"=>nil, "title"=>"Debugging Rails", "body"=>"I'm learning how to print in logs!!!",
 "published"=>false, "created_at"=>nil}
Article should be valid: true
  Article Create (0.000443)   INSERT INTO "articles" ("updated_at", "title", "body", "published",
 "created_at") VALUES('2008-09-08 14:52:54', 'Debugging Rails',
 'I''m learning how to print in logs!!!', 'f', '2008-09-08 14:52:54')
The article was saved and now the user is going to be redirected...
Redirected to # Article:0x20af760>
Completed in 0.01224 (81 reqs/sec) | DB: 0.00044 (3%) | 302 Found [http://localhost/articles]
```

La adición de logs como éste hace que sea fácil buscar un comportamiento inesperado o inusual en sus logs. Si agrega extra logs, asegúrese de hacer un uso razonable de los niveles de logs para evitar llenar sus logs de producción con trivialidades inútiles.

### 2.4  Logging marcado

Cuando se ejecutan aplicaciones multiusuario y de varias cuentas, a menudo es útil poder filtrar los logs mediante algunas reglas personalizadas. `TaggedLogging` en Active Support le ayuda a hacer exactamente eso mediante el tagueado de líneas de logs con subdominios, ids de solicitud y cualquier otra cosa que ayude a depurar dichas aplicaciones.

```ruby
logger = ActiveSupport::TaggedLogging.new(Logger.new(STDOUT))
logger.tagged("BCX") { logger.info "Stuff" }                            # Logs "[BCX] Stuff"
logger.tagged("BCX", "Jason") { logger.info "Stuff" }                   # Logs "[BCX] [Jason] Stuff"
logger.tagged("BCX") { logger.tagged("Jason") { logger.info "Stuff" } } # Logs "[BCX] [Jason] Stuff"
```

### 2.5 Impacto de los logs en el performance

El log tendrá siempre un pequeño impacto en el rendimiento de su aplicación de Rails, en particular al iniciar sesión en el disco. Además, hay algunas sutilezas:

El uso del nivel de `:debug` tendrá una penalización de rendimiento mayor que `:fatal`, ya que se está evaluando y escribiendo un número mucho mayor de cadenas en la salida del log \(por ejemplo, disco\).

Otra posible falla es demasiadas llamadas a `Logger` en tu código:

```ruby
logger.debug "Person attributes hash: #{@person.attributes.inspect}"
```

En el ejemplo anterior, habrá un impacto en el rendimiento incluso si el nivel de salida permitido no incluye debug. La razón es que Ruby tiene que evaluar estas cadenas, lo que incluye instanciar el objeto String, lo que es algo pesado y debe interpolar las variables. Por lo tanto, se recomienda pasar bloques a los métodos de logger, ya que sólo se evalúen si el nivel de salida es el mismo que el nivel permitido \(es decir, carga lenta\). El mismo código reescrito sería:

```ruby
logger.debug {"Person attributes hash: #{@person.attributes.inspect}"}
```

El contenido del bloque, y por lo tanto la interpolación de cadena, sólo se evalúan si debug está habilitado. Este ahorro de rendimiento sólo se nota realmente con grandes cantidades de logs, pero es una buena práctica por emplear.









