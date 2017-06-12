# 5- Carga de archivos

Una tarea común es cargar algún tipo de archivo, ya sea una imagen de una persona o un archivo CSV que contenga datos para procesar. Lo más importante a recordar con las subidas de archivos es que la codificación de la forma renderizada **DEBE** establecerse en "`multipart/form-data`". Si utiliza `form_for`, esto se hace automáticamente. Si utiliza `form_tag`, debe configurarlo usted mismo, como en el ejemplo siguiente.

```ruby
<%= form_tag({action: :upload}, multipart: true) do %>
  <%= file_field_tag 'picture' %>
<% end %>
 
<%= form_for @person do |f| %>
  <%= f.file_field :picture %>
<% end %>
```

Rails proporciona el par habitual de helpers: el barebones `file_field_tag` y el orientado a modelos `file_field`. La única diferencia con otros helpers es que no se puede establecer un valor predeterminado para los inputs de archivos ya que esto no tendría ningún significado. Como es de esperar en el primer caso el archivo subido está en `params[:picture] `y en el segundo caso en `params[:person][:picture]`.



### 5.1 Lo que se sube

El objeto en el hash `params` es una instancia de una subclase de `IO`. Dependiendo del tamaño del archivo subido puede ser de hecho un `StringIO` o una instancia de Archivo respaldado por un archivo temporal. En ambos casos, el objeto tendrá un atributo `original_filename` que contiene el nombre del archivo en el equipo del usuario y un atributo `content_type` que contiene el tipo `MIME` del archivo subido. El fragmento siguiente guarda el contenido cargado en `#{Rails.root}/public/uploads` con el mismo nombre que el archivo original \(suponiendo que el formulario era el del ejemplo anterior\).

```ruby
def upload
  uploaded_io = params[:person][:picture]
  File.open(Rails.root.join('public', 'uploads', uploaded_io.original_filename), 'wb') do |file|
    file.write(uploaded_io.read)
  end
end
```

Una vez que se ha cargado un archivo, hay una multitud de tareas potenciales, desde: dónde almacenar los archivos \(en disco, Amazon S3, etc.\) y asociarlos con modelos para cambiar el tamaño de los archivos de imagen y generar miniaturas. Las complejidades de esto están más allá del alcance de esta guía, pero hay varias bibliotecas diseñadas para ayudar con ellas. Dos de los más conocidos son CarrierWave y Paperclip.

> Si el usuario no ha seleccionado un archivo, el parámetro correspondiente será una cadena vacía.





### 5.2 Tratamiento con Ajax

A diferencia de otros formularios, crear un archivo de carga asíncrona no es tan sencillo como proporcionar un `form_for` con `remote :true`. Con un formulario de Ajax la serialización se hace por JavaScript que se ejecuta dentro del navegador y ya que JavaScript no puede leer archivos de su disco duro el archivo no se puede cargar. La solución más común es utilizar un iframe invisible que sirva como objetivo para la presentación del formulario.

