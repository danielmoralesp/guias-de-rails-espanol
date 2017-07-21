# 9- Encabezados predeterminados

Cada respuesta HTTP de su aplicación Rails recibe los siguientes encabezados de seguridad predeterminados.

```
config.action_dispatch.default_headers = {
  'X-Frame-Options' => 'SAMEORIGIN',
  'X-XSS-Protection' => '1; mode=block',
  'X-Content-Type-Options' => 'nosniff'
}
```

Puede configurar encabezados predeterminados en `config/application.rb`.

```
config.action_dispatch.default_headers = {
  'Header-Name' => 'Header-Value',
  'X-Frame-Options' => 'DENY'
}
```

O usted puede quitarlos.

```ruby
config.action_dispatch.default_headers.clear
```

Aquí hay una lista de encabezados comunes:

* **X-Frame-Options**: 'SAMEORIGIN' en Rails por defecto - permite enmarcar en el mismo dominio. Póngalo en 'DENY' para negar el framing en absoluto o 'ALLOWALL' si desea permitir el framing para todo el sitio web.
* **X-XSS-Protection**: '1; Mode = block 'en Rails por defecto - use XSS Auditor y bloquee la página si se detecta un ataque XSS. Establecer a '0;' Si desea desactivar XSS Auditor \(útil si los scripts de contenido de respuesta de los parámetros de solicitud\)
* **X-Content-Type-Options**: 'nosniff' en Rails de forma predeterminada: impide que el navegador adivine el tipo MIME de un archivo.
* **X-Content-Security-Policy**: Un potente mecanismo para controlar qué sitios se pueden cargar ciertos tipos de contenido desde alli
* **Access-Control-Allow-Origin**: Se utiliza para controlar qué sitios se les permite eludir las mismas políticas de origen y enviar peticiones de origen cruzado.
* **Strict-Transport-Security**: se utiliza para controlar si el navegador sólo tiene acceso a un sitio a través de una conexión segura



