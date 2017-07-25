# 8- Hacer de su libreria o gema un preprocesador

Sprockets utiliza procesadores, transformadores, compresores y exportadores para ampliar la funcionalidad de los Sprockets. Echa un vistazo a [extending Sprockets](https://github.com/rails/sprockets/blob/master/guides/extending_sprockets.md) para obtener más información. Aquí hemos registrado un preprocesador para agregar un comentario al final de los archivos de `text/css` \(`.css`\).

```ruby
module AddComment
  def self.call(input)
    { data: input[:data] + "/* Hello From my sprockets extension */" }
  end
end
```

Ahora que tiene un módulo que modifica los datos de entrada, es el momento de registrarlo como un preprocesador para su mime type.

```ruby
Sprockets.register_preprocessor 'text/css', AddComment
```



