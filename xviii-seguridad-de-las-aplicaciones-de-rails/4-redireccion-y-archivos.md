# 4- Redirección y archivos

Otra clase de vulnerabilidades de seguridad rodea el uso de la redirección y los archivos en las aplicaciones web.

### 4.1 Redirección

> La redirección en una aplicación web es una herramienta de cracker subestimada: no sólo el atacante puede enviar al usuario a un sitio web falso, sino que también puede crear un ataque autónomo.

Siempre que se permite al usuario pasar \(partes de\) la URL para la redirección, es posiblemente vulnerable. El ataque más obvio sería redirigir a los usuarios a una aplicación web falsa que se ve y se siente exactamente como la original. Este llamado ataque phishing funciona enviando un enlace no sospechoso en un correo electrónico a los usuarios, inyectando el enlace por _XSS_ en la aplicación web o poniendo el enlace en un sitio externo. Es poco sospechoso, porque el enlace comienza con la URL de la aplicación web y la dirección URL del sitio malintencionado está oculta en el parámetro de redirección: `http://www.example.com/site/redirect?to=www.attacker.com` . He aquí un ejemplo de una acción heredada:

```ruby
def legacy
  redirect_to(params.update(action:'main'))
end
```

Esto redireccionará al usuario a la acción principal si intenta acceder a una acción heredada. La intención era preservar los parámetros de la URL a la acción heredada y pasarlos a la acción principal. Sin embargo, puede ser explotado por el atacante si incluyeron una clave de host en la URL:

```html
http://www.example.com/site/legacy?param1=xy&param2=23&host=www.attacker.com
```

Si se encuentra al final de la URL, difícilmente se notará y redireccionará al usuario al host `attacker.com`. Una contramedida simple sería incluir sólo los parámetros esperados en una acción heredada \(de nuevo un enfoque de lista blanca, en lugar de eliminar parámetros inesperados\). Y si rediriges a una URL, compruébala con una lista blanca o una expresión regular.

### 4.1.1 XSS auto-contenido

Otra redirección y el ataque auto-contenido XSS funciona en Firefox y Opera mediante el uso del protocolo de datos. Este protocolo muestra su contenido directamente en el navegador y puede ser cualquier cosa desde HTML o JavaScript a imágenes enteras:

`data:text/html;base64,PHNjcmlwdD5hbGVydCgnWFNTJyk8L3NjcmlwdD4K`

Este ejemplo es un JavaScript codificado en Base64 que muestra un cuadro de mensaje simple. En una URL de redirección, un atacante podría redirigir a esta URL con el código malicioso en ella. Como contramedida, no permita que el usuario proporcione \(partes de\) la URL a la que se redirigirá.

### 4.2 Cargas de archivos

Asegúrese de que las subidas de archivos no sobrescriben archivos importantes y procesen los archivos multimedia de forma asíncrona.

Muchas aplicaciones web permiten a los usuarios cargar archivos. Los nombres de archivo, que el usuario puede elegir \(en parte\), siempre deben ser filtrados, ya que un atacante podría usar un nombre de archivo malicioso para sobrescribir cualquier archivo del servidor. Si almacena subidas de archivos en `/var/www/uploads `y el usuario introduce un nombre de archivo como "`../../../etc/passwd`", puede sobrescribir un archivo importante. Por supuesto, el intérprete de Ruby necesitaría los permisos apropiados para hacerlo - una razón más para ejecutar servidores web, servidores de bases de datos y otros programas como un usuario Unix menos privilegiado.

Al filtrar los nombres de los archivos de entrada del usuario, no intente eliminar partes malintencionadas. Piense en una situación en la que la aplicación web quita todo el "`../`" en un nombre de archivo y un atacante utiliza una cadena como "`.... //`" - el resultado será "`../`". Lo mejor es utilizar un enfoque de lista blanca, que comprueba la validez de un nombre de archivo con un conjunto de caracteres aceptados. Esto se opone a un enfoque de lista negra que intenta eliminar caracteres no permitidos. En caso de que no sea un nombre de archivo válido, rechácelo \(o reemplace los caracteres no aceptados\), pero no los elimine. Aquí está el nombre de archivo `sanitizer` del plugin `attachment_fu`:

```ruby
def sanitize_filename(filename)
  filename.strip.tap do |name|
    # NOTE: File.basename doesn't work right with Windows paths on Unix
    # get only the filename, not the whole path
    name.sub! /\A.*(\\|\/)/, ''
    # Finally, replace all non alphanumeric, underscore
    # or periods with underscore
    name.gsub! /[^\w\.\-]/, '_'
  end
end
```

Una desventaja significativa del procesamiento síncrono de subidas de archivos \(como con el plugin `attachment_fu` que puede subir imágenes\), es su vulnerabilidad a los ataques de denegación de servicio. Un atacante puede iniciar de forma sincronizada cargas de archivos de imagen desde muchas computadoras, lo que aumenta la carga del servidor y eventualmente se bloquea o bloquea el servidor.

La solución a esto es mejor procesar los archivos de medios de forma asíncrona: Guarde el archivo de medios y programe una solicitud de procesamiento en la base de datos. Un segundo proceso manejará el proceso del archivo en el background.

### 4.3 Código ejecutable en cargas de archivos

> El código fuente en los archivos subidos puede ser ejecutado cuando se coloca en directorios específicos. No coloque cargas de archivos en el directorio de Rails `/public` si es el directorio inicial de Apache.

El popular servidor web Apache tiene una opción denominada `DocumentRoot`. Este es el directorio de inicio del sitio web, todo en este árbol de directorio será servido por el servidor web. Si hay archivos con una extensión de nombre de archivo determinada, el código en ella se ejecutará cuando se solicite \(podría requerir algunas opciones para establecer\). Ejemplos de esto son archivos `PHP` y `CGI`. Ahora piensa en una situación en la que un atacante carga un archivo "`file.cgi`" con código en él, que se ejecutará cuando alguien descargue el archivo.

Si su Apache `DocumentRoot` apunta al directorio de Rails `/public`, no coloque archivos cargados en él, almacene los archivos al menos un nivel hacia abajo.

### 4.4 Descargas de archivos

Asegúrese de que los usuarios no pueden descargar archivos arbitrarios.

Del mismo modo que tiene que filtrar los nombres de archivo para las subidas, tiene que hacerlo para las descargas. El método `send_file()` envía archivos desde el servidor al cliente. Si utiliza un nombre de archivo, que el usuario ingresó, sin filtrar, el puede descargar cualquier archivo:

```ruby
send_file('/var/www/uploads/' + params[:filename])
```

Simplemente pase un nombre de archivo como este "`../../../etc/passwd`" para descargar la información de inicio de sesión del servidor. Una solución simple contra esto es comprobar que el archivo solicitado está en el directorio esperado:

```ruby
basename = File.expand_path(File.join(File.dirname(__FILE__), '../../files'))
filename = File.expand_path(File.join(basename, @file.public_filename))
raise if basename !=
     File.expand_path(File.join(File.dirname(filename), '../../../'))
send_file filename, disposition: 'inline'
```

Otro enfoque \(adicional\) es almacenar los nombres de archivo en la base de datos y nombrar los archivos en el disco después de los ids en la base de datos. Éste es también un buen acercamiento para evitar el posible código en un archivo subido para ser ejecutado. El plugin `attachment_fu` hace esto de una manera similar.













