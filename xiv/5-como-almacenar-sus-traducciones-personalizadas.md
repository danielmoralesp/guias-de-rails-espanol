# 5- Cómo almacenar sus traducciones personalizadas

El backend simple incluido con Active Support le permite almacenar traducciones en formato Ruby y YAML.

Por ejemplo, un Ruby Hash que proporciona traducciones puede verse así:

```ruby
{
  pt: {
    foo: {
      bar: "baz"
    }
  }
}
```

El archivo YAML equivalente sería así:

```ruby
pt:
  foo:
    bar: baz
```

Como puede ver, en ambos casos la clave de nivel superior es la configuración regional. `:foo` es una clave de espacio de nombres y `:bar` es la clave para la traducción "`baz`".

Este es un ejemplo "real" del archivo YAML de Active Support `en.yml` translations:

```ruby
en:
  date:
    formats:
      default: "%Y-%m-%d"
      short: "%b %d"
      long: "%B %d, %Y"
```

Por lo tanto, todas las siguientes consultas equivalentes devolverán el formato de fecha `:short` "`%b %d`":

```ruby
I18n.t 'date.formats.short'
I18n.t 'formats.short', scope: :date
I18n.t :short, scope: 'date.formats'
I18n.t :short, scope: [:date, :formats]
```

Por lo general, recomendamos usar YAML como un formato para almacenar las traducciones. Hay casos, sin embargo, donde desea almacenar lambdas de Ruby como parte de sus datos de configuración regional, por ejemplo para formatos de fechas especiales.

