# 3- Internacionalización y localización

¡Ok! Ahora ha _**inicializado**_ la compatibilidad con I18n para su aplicación de Ruby on Rails y le ha indicado qué idioma usar y cómo preservarla entre las solicitudes.

Luego necesitamos _**internacionalizar**_ nuestra aplicación abstrayendo cada elemento específico de la configuración regional. Finalmente, necesitamos _**localizarlo**_ proporcionando las traducciones necesarias para estos resúmenes.

Dado el siguiente ejemplo:

```ruby
# config/routes.rb
Rails.application.routes.draw do
  root to: "home#index"
end
```

```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  before_action :set_locale

  def set_locale
    I18n.locale = params[:locale] || I18n.default_locale
  end
end
```

```ruby
# app/controllers/home_controller.rb
class HomeController < ApplicationController
  def index
    flash[:notice] = "Hello Flash"
  end
end
```

```ruby
# app/views/home/index.html.erb
<h1>Hello World</h1>
<p><%= flash[:notice] %></p>
```

![](/assets/Captura de pantalla 2017-07-12 a las 2.21.42 p.m..png)

### 3.1 Abstrayendo el código a localizar

Hay dos cadenas en nuestro código que están en inglés y que los usuarios renderizarán en nuestra respuesta \("Hello Flash" y "Hello World"\). Para internacionalizar este código, estas cadenas deben ser reemplazadas por llamadas al ayudante  `#t` de Rails con una clave apropiada para cada cadena:

```ruby
# app/controllers/home_controller.rb
class HomeController < ApplicationController
  def index
    flash[:notice] = t(:hello_flash)
  end
end
```

```ruby
# app/views/home/index.html.erb
<h1><%=t :hello_world %></h1>
<p><%= flash[:notice] %></p>
```

Ahora, cuando se muestre esta vista, mostrará un mensaje de error que indica que faltan las traducciones de las claves `:hello_world` y `:hello_flash`.

![](/assets/Captura de pantalla 2017-07-12 a las 2.35.23 p.m..png)

> Rails añade un método auxiliar `t`\(translate\) a sus vistas de modo que no necesite deletrear `I18n.t` todo el tiempo. Además, este ayudante capturará las traducciones perdidas y envolverá el mensaje de error resultante en un `<span class = "translation_missing">`.

### 3.2 Proporcionar traducciones para cadenas internacionalizadas

Agregue las traducciones faltantes a los archivos del diccionario de traducción:

```ruby
# config/locales/en.yml
en:
  hello_world: Hello world!
  hello_flash: Hello flash!

# config/locales/pirate.yml
pirate:
  hello_world: Ahoy World
  hello_flash: Ahoy Flash
```

vamos aqui el jueves 13 de julio

