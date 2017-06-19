# 11- Autenticaciones HTTP

Rails viene con dos mecanismos de autenticación HTTP integrados:

* Autenticación básica
* Autenticación Digest

### 11.1 Autenticación básica HTTP

La autenticación `HTTP` básica es un esquema de autenticación que es compatible con la mayoría de los navegadores y otros clientes `HTTP`. Por ejemplo, considere una sección de administración que sólo estará disponible introduciendo un nombre de usuario y una contraseña en la ventana de diálogo `HTTP` básica del navegador. El uso de la autenticación integrada es bastante sencilla y sólo requiere que utilice un método, `http_basic_authenticate_with`.

```ruby
class AdminsController < ApplicationController
  http_basic_authenticate_with name: "humbaba", password: "5baa61e4"
end
```

Con esto en su lugar, puede crear controladores de namespaces que hereden de `AdminsController`. El filtro se ejecutará para todas las acciones en esos controladores, protegiéndolos con la autenticación básica `HTTP`.

### 11.2 Autenticación HTTP Digest

La autenticación `HTTP` digest es superior a la autenticación básica, ya que no requiere que el cliente envíe una contraseña sin cifrar a través de la red \(aunque la autenticación `HTTP` básica es segura en `HTTPS`\). El uso de la autenticación digest con Rails es bastante sencillo y sólo requiere el uso de un método, `authenticate_or_request_with_http_digest`.

```ruby
class AdminsController < ApplicationController
  USERS = { "lifo" => "world" }
 
  before_action :authenticate
 
  private
 
    def authenticate
      authenticate_or_request_with_http_digest do |username|
        USERS[username]
      end
    end
end
```

Como se ve en el ejemplo anterior, el bloque `authenticate_or_request_with_http_digest` sólo tiene un argumento: el nombre de usuario. Y el bloque devuelve la contraseña. Devolver `false` o `nil` desde el  `authenticate_or_request_with_http_digest` causará un error de autenticación.



