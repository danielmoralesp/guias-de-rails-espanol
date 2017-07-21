# 4- Depuración con la gema web-console

Web Console es un poco como byebug, pero se ejecuta en el navegador. En cualquier página que esté desarrollando, puede solicitar una consola en el contexto de una vista o de un controlador. La consola se renderizará al lado de su contenido HTML.

### 4.1 Consola

Dentro de cualquier acción o vista del controlador, puede invocar la consola llamando al método `console`.

Por ejemplo, en un controlador:

```ruby
class PostsController < ApplicationController
  def new
    console
    @post = Post.new
  end
end
```

O en una vista:

```html
<% console %>
 
<h2>New Post</h2>
```

Esto hará que se renderice `console` dentro de su vista. No necesita preocuparse por la ubicación de la llamada a la consola; No se renderizará en el lugar de su invocación sino junto a su contenido HTML.

La consola ejecuta código Ruby puro: Puede definir e instanciar clases personalizadas, crear nuevos modelos e inspeccionar variables.

> Sólo se puede renderizar una consola por solicitud. De lo contrario, `console-web` generará un error en la segunda invocación de la consola.

### 4.2 Inspección de variables

Puede invocar las variables de instancia para listar todas las variables de instancia disponibles en su contexto. Si desea listar todas las variables locales, puede hacerlo con `local_variables`.

### 4.3 Ajustes

* `config.web_console.whitelisted_ips`: Lista autorizada de direcciones y redes IPv4 o IPv6 \(valores por defecto:  `127.0.0.1/8, ::1`\).
* `config.web_console.whiny_requests`: Registra un mensaje cuando se evita un renderizado de consola \(defaults: `true`\).

Dado que console-web evalúa el código Ruby en forma remota en el servidor, no intente utilizarlo en producción.













