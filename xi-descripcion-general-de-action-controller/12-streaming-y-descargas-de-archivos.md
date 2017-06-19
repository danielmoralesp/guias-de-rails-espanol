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



