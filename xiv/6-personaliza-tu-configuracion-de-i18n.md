# 6- Personaliza tu configuración de I18n

### 6.1 Uso de diferentes backends

Por varias razones, el back-end Simple incluido con Active Support solo hace lo "más sencillo que posiblemente podría funcionar" para Ruby on Rails3 ... lo que significa que sólo se garantiza que funcione para el inglés y como un efecto secundario, los idiomas que son muy Similar al inglés. Además, el backend simple sólo es capaz de leer traducciones pero no puede guardarlas dinámicamente en ningún formato.

Sin embargo, eso no significa que esté atrapado con estas limitaciones. La gema Ruby I18n hace que sea muy fácil intercambiar la implementación simple de backend con algo más que se adapte mejor a sus necesidades. P.ej. Usted podría intercambiarlo con el backend estático de Globalize:

```ruby
I18n.backend = Globalize::Backend::Static.new
```

También puede utilizar el backend de cadena para encadenar múltiples backends juntos. Esto es útil cuando se desea usar traducciones estándar con un backend simple, pero almacenar traducciones de aplicaciones personalizadas en una base de datos u otros backends. Por ejemplo, puede utilizar el backend de Active Record y volver al backend simple \(por defecto\):

```ruby
I18n.backend = I18n::Backend::Chain.new(I18n::Backend::ActiveRecord.new, I18n.backend)
```

### 6.2 Uso de diferentes manejadores de excepciones

La API I18n define las siguientes excepciones que serán generadas por backends cuando se produzcan las condiciones inesperadas correspondientes:

```ruby
MissingTranslationData       # no translation was found for the requested key
InvalidLocale                # the locale set to I18n.locale is invalid (e.g. nil)
InvalidPluralizationData     # a count option was passed but the translation data is not suitable for pluralization
MissingInterpolationArgument # the translation expects an interpolation argument that has not been passed
ReservedInterpolationKey     # the translation contains a reserved interpolation variable name (i.e. one of: scope, default)
UnknownFileType              # the backend does not know how to handle a file type that was added to I18n.load_path
```

La API I18n captura todas estas excepciones cuando se lanzan en el backend y las pasan al método `default_exception_handler`. Este método volverá a aumentar todas las excepciones, excepto las excepciones de `MissingTranslationData`. Cuando se ha detectado una excepción `MissingTranslationData`, devolverá la cadena de mensaje de error de la excepción que contiene la clave o el ámbito que faltan.

La razón de esto es que durante el desarrollo en el que normalmente desea que sus vistas se usen a pesar de que falte una traducción.

Sin embargo, en otros contextos puede que desee cambiar este comportamiento. P.ej. La manipulación de excepciones por defecto no permite capturar fácilmente las traducciones perdidas durante las pruebas automatizadas. Para este propósito se puede especificar un manejador de excepciones diferente. El manejador de excepciones especificado debe ser un método en el módulo I18n o una clase con el método `#call`:

```ruby
module I18n
  class JustRaiseExceptionHandler < ExceptionHandler
    def call(exception, locale, key, options)
      if exception.is_a?(MissingTranslationData)
        raise exception.to_exception
      else
        super
      end
    end
  end
end
 
I18n.exception_handler = I18n::JustRaiseExceptionHandler.new
```

Esto volvería a levantar sólo la excepción `MissingTranslationData`, pasando todas las demás entradas al controlador de excepciones predeterminado.

Sin embargo, si está utilizando `I18n::Backend::Pluralization` este manejador también levantará la excepción `I18n::MissingTranslationData: translation missing: en.i18n.plural.rule` que normalmente se debe ignorar para volver a la regla de pluralización por defecto para el idioma Inglés . Para evitar esto, puede usar un chequeo adicional para la clave de traducción:

```ruby
if exception.is_a?(MissingTranslationData) && key.to_s != 'i18n.plural.rule'
  raise exception.to_exception
else
  super
end
```

Otro ejemplo donde el comportamiento por defecto es menos deseable, es el Rails `TranslationHelper` que proporciona el método `#t` \(así como `#translate`\). Cuando se produce una excepción `MissingTranslationData` en este contexto, el ayudante envuelve el mensaje en un intervalo con la clase CSS `translation_missing`.

Para ello, el helper fuerza a `I18n#translate` a generar excepciones, independientemente del manejador de excepciones que se defina al establecer la opción `:raise`:

```ruby
I18n.t :foo, raise: true # always re-raises exceptions from the backend
```



