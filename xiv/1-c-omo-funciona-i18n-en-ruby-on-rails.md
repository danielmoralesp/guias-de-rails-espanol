# 1- Cómo funciona I18n en Ruby on Rails

La internacionalización es un problema complejo. Los lenguajes naturales difieren de muchas maneras \(por ejemplo, en las reglas de pluralización\) que es difícil proporcionar herramientas para resolver todos los problemas a la vez. Por ello, la API de I18n se centra en:

* Proporcionar soporte para idiomas ingleses y similares "out of the box".
* Por lo que es fácil de personalizar y ampliar todo para otros idiomas

Como parte de esta solución, **cada cadena estática en el framework de Rails**, por ejemplo los mensajes de validación de Active Record, y los formatos de fecha y hora, `se han internacionalizado`. La _localización \(localization\)_ de una aplicación de Rails significa definir valores traducidos para estas cadenas en los idiomas deseados.

### 1.1 La Arquitectura General de la Biblioteca

Por lo tanto, la gema Ruby I18n se divide en dos partes:

* La API pública del framework i18n - un módulo Ruby con métodos públicos que definen cómo funciona la biblioteca
* Un backend predeterminado \(que se nombra intencionalmente Backend simple\) que implementa estos métodos

Como usuario siempre debes acceder solo a los métodos públicos en el módulo I18n, pero es útil conocer las capacidades del backend.

> Es posible intercambiar el backend simple enviado con un más potente, que almacenaría datos de la traducción en una base de datos relacional, un diccionario de GetText, o similar. Consulte la sección Utilización de diferentes backends en la sección 6.1

### 1.2 La API pública I18n

Los métodos más importantes de la API I18n son:

```ruby
translate # Buscar traducciones de texto
localize  # Localiza objetos de fecha y hora en formatos locales
```

Estos tienen los alias \#t y \#l para que pueda utilizarlos de esta manera:

```ruby
I18n.t 'store.title'
I18n.l Time.now
```

También hay lectores y escritores de atributos para los siguientes atributos:

```ruby
load_path                 # Anuncie sus archivos de traducción personalizados
locale                    # Obtener y establecer el locale actual
default_locale            # Obtener y establecer la configuración regional predeterminada
available_locales         # Posibilidades de listas blancas disponibles para la aplicación
enforce_available_locales # Aplicar la lista blanca de configuración regional (true o false)
exception_handler         # Utilizar un manejador de excepción diferente
backend                   # Utilizar un backend diferente
```

Así que internacionalizaremos una sencilla aplicación de Rails desde el principio en los próximos capítulos.

