# 12- Streaming y descargas de archivos

A veces es posible que desee enviar un archivo al usuario en lugar de renderizar una página `HTML`. Todos los controladores de Rails tienen los métodos `send_data` y `send_file`, que transmitirán los datos al cliente. `send_file` es un método de conveniencia que le permite proporcionar el nombre de un archivo en el disco y transmitirá el contenido de ese archivo para usted.

Para transmitir datos al cliente, utilice `send_data`:

```ruby
require "prawn"
class ClientsController < ApplicationController
  # Generates a PDF document with information on the client and
  # returns it. The user will get the PDF as a file download.
  def download_pdf
    client = Client.find(params[:id])
    send_data generate_pdf(client),
              filename: "#{client.name}.pdf",
              type: "application/pdf"
  end

  private

    def generate_pdf(client)
      Prawn::Document.new do
        text client.name, align: :center
        text "Address: #{client.address}"
        text "Email: #{client.email}"
      end.render
    end
end
```

La acción `download_pdf` en el ejemplo anterior llamará a un método privado que realmente genera el documento PDF y lo devuelve como una cadena. Esta cadena se transmitirá al cliente como una descarga de archivos y se le sugerirá al usuario un nombre de archivo. A veces, al transferir archivos al usuario, es posible que no desee que descarguen el archivo. Tomar imágenes, por ejemplo, que se pueden incrustar en páginas HTML. Para decirle al navegador que un archivo no está destinado a ser descargado, puede establecer la opción `:disposition` en "`inline`". El valor opuesto y el valor predeterminado para esta opción es "`attachment`".

### 12.1 Envío de archivos

Si desea enviar un archivo que ya existe en el disco, utilice el método `send_file`.

```ruby
class ClientsController < ApplicationController
  # Stream a file that has already been generated and stored on disk.
  def download_pdf
    client = Client.find(params[:id])
    send_file("#{Rails.root}/files/clients/#{client.id}.pdf",
              filename: "#{client.name}.pdf",
              type: "application/pdf")
  end
end
```

Esto leerá y transmitirá el archivo 4kB en ese momento, evitando cargar todo el archivo en la memoria a la vez. Puede desactivar la transmisión con la opción `:stream` o ajustar el tamaño del bloque con la opción `:buffer_size`.

Si `:type` no se especifica, se adivinará desde la extensión de archivo especificada en `:filename`. Si el tipo de contenido no está registrado para la extensión, se utilizará `application/octet-stream`.

> Tenga cuidado al utilizar los datos procedentes del cliente \(params, cookies, etc.\) para localizar el archivo en disco, ya que se trata de un riesgo de seguridad que podría permitirle a alguien acceder a archivos a los que no están destinados.

> No se recomienda que transmita archivos estáticos a través de Rails si puede guardarlos en una carpeta pública en su servidor web. Es mucho más eficiente dejar que el usuario descargue el archivo directamente usando Apache u otro servidor web, evitando que la petición pase innecesariamente por toda la pila de Rails.

### 12.2 Descargas RESTful

Aunque `send_data` funciona correctamente, si no está creando una aplicación RESTful que tenga acciones separadas para descargas de archivos, normalmente no es necesario. En la terminología REST, el archivo PDF del ejemplo anterior puede considerarse simplemente otra representación del recurso del cliente. Rails proporciona una forma fácil y bastante elegante de hacer "descargas RESTful". Así es como puede volver a escribir el ejemplo para que la descarga de PDF sea una parte de la acción `show`, sin ningún streaming:

```ruby
class ClientsController < ApplicationController
  # The user can request to receive this resource as HTML or PDF.
  def show
    @client = Client.find(params[:id])
 
    respond_to do |format|
      format.html
      format.pdf { render pdf: generate_pdf(@client) }
    end
  end
end
```

Para que este ejemplo funcione, debe agregar el tipo MIME PDF a Rails. Esto se puede hacer agregando la siguiente línea al archivo `config/initializers/mime_types.rb`:

```ruby
Mime::Type.register "application/pdf", :pdf
```

> Los archivos de `config` no se vuelven a cargar en cada solicitud, por lo que debe reiniciar el servidor para que sus cambios surtan efecto.

Ahora el usuario puede solicitar para obtener una versión en PDF de un cliente simplemente añadiendo "`.pdf`" a la URL:

```ruby
GET /clients/1.pdf
```

### 12.3 Transmisión en directo de datos arbitrarios

Rails le permite transmitir más que sólo archivos. De hecho, puedes transmitir todo lo que quieras en un objeto de respuesta. El módulo `ActionController::Live` le permite crear una conexión persistente con un navegador. Usando este módulo, usted podrá enviar datos arbitrarios al navegador en puntos específicos en el tiempo.

#### 12.3.1 Incorporación de Live Streaming

Incluyendo `ActionController::Live` dentro de su clase controlador proporcionará todas las acciones dentro del controlador la capacidad de transmitir datos. Puede mezclar el módulo de la siguiente manera:

```ruby
class MyController < ActionController::Base
  include ActionController::Live
 
  def stream
    response.headers['Content-Type'] = 'text/event-stream'
    100.times {
      response.stream.write "hello world\n"
      sleep 1
    }
  ensure
    response.stream.close
  end
end
```

El código anterior mantendrá una conexión persistente con el navegador y enviará 100 mensajes de "`hola mundo \ n`", cada segundo.

Hay un par de cosas a notar en el ejemplo anterior. Necesitamos cerciorarnos de cerrar el flujo de la respuesta. Olvidar cerrar el stream dejará el socket abierto para siempre. También tenemos que establecer el tipo de contenido en `text/event-stream` antes de escribirlo en el flujo de respuesta. Esto se debe a que los encabezados no se pueden escribir después de que se ha enviado la respuesta \(cuando `response.committed?` devuelve un valor verdadero\), que se produce cuando se escribe o se confirma el flujo de respuesta.

#### 12.3.2 Ejemplo de uso

Supongamos que usted estaba haciendo una máquina de Karaoke y un usuario quiere obtener las letras de una canción en particular. Cada canción tiene un número particular de líneas y cada línea toma el tiempo `num_beats` para terminar el canto.

Si queríamos devolver las letras en modo Karaoke \(sólo enviar la línea cuando el cantante haya terminado la línea anterior\), entonces podríamos usar `ActionController::Live` de la siguiente manera:

```ruby
class LyricsController < ActionController::Base
  include ActionController::Live
 
  def show
    response.headers['Content-Type'] = 'text/event-stream'
    song = Song.find(params[:id])
 
    song.each do |line|
      response.stream.write line.lyrics
      sleep line.num_beats
    end
  ensure
    response.stream.close
  end
end
```

El código anterior envía la siguiente línea sólo después de que el cantante haya completado la línea anterior.

#### 12.3.3 Consideraciones del streaming

La transmisión de datos arbitrarios es una herramienta extremadamente poderosa. Como se muestra en los ejemplos anteriores, puede elegir cuándo y qué enviar a través de un flujo de respuesta. Sin embargo, también debe tener en cuenta lo siguiente:

* Cada flujo de respuesta crea un nuevo subproceso y copia sobre las variables locales de subproceso del subproceso original. Tener demasiadas variables locales de subproceso puede afectar negativamente el rendimiento. Del mismo modo, un gran número de hilos también pueden dificultar el rendimiento.
* Si no se cierra el flujo de respuesta se deja el socket correspondiente abierto para siempre. Asegúrese de llamar `close` cada vez que utilice un flujo de respuesta.
* Los servidores `WEBrick` almacenan todas las respuestas, por lo que incluir `ActionController::Live` no funcionará. Debe utilizar un servidor web que no almacena automáticamente las respuestas.



