# 2- Extensiones a todos los objetos

### 2.1 `blank?` Y `present?`

Los valores siguientes se consideran en blanco en una aplicación de Rails:

* `nil` y `false`,
* Strings compuestos sólo de espacios en blanco \(ver nota más abajo\),
* Arrays h hashes vacíos
* Cualquier otro objeto que responde a `empty?` Y está vacío.

> El predicado de las cadenas usa los caracteres Unicode-aware para la `class[:space:]`, por lo que por ejemplo U+2029 \(separador de párrafos\) se considera como espacio en blanco.
>
> Tenga en cuenta que los números no se mencionan. En particular, 0 y 0,0 no están en blanco.

Por ejemplo, este método de `ActionController::HttpAuthentication::Token::ControllerMethods` utiliza `blank?` Para comprobar si un token está presente:

```ruby
def authenticate(controller, &login_procedure)
  token, options = token_and_options(controller.request)
  unless token.blank?
    login_procedure.call(token, options)
  end
end
```

El método `present?` Es equivalente a `!blank?`. Este ejemplo se toma de `ActionDispatch::Http::Cache::Response`:

```ruby
def set_conditional_cache_control!
  return if self["Cache-Control"].present?
  ...
end
```

Se define en `active_support/core_ext/object/blank.rb`.

### 2.2 presence

El método de `presence` devuelve su receptor si está `present?`, y `nil` en caso contrario. Es útil para expresiones como esta:

```ruby
host = config[:host].presence || 'localhost'
```

> Se define en `active_support/core_ext/object/blank.rb`.

### 2.3 duplicable?

En Ruby 2.4 la mayoría de los objetos pueden ser duplicados vía `dup` o `clone` excepto los métodos y ciertos números. Aunque Ruby 2.2 y 2.3 no pueden duplicar `nil`, `false`, `true` y símbolos, así como instancias de `Float`, `Fixnum` y `Bignum`.

```ruby
"foo".dup           # => "foo"
"".dup              # => ""
1.method(:+).dup    # => TypeError: allocator undefined for Method
Complex(0).dup      # => TypeError: can't copy Complex
```

Active Support proporciona `duplicable?` para consultar un objeto acerca de esto:

```ruby
"foo".duplicable?           # => true
"".duplicable?              # => true
Rational(1).duplicable?     # => false
Complex(1).duplicable?      # => false
1.method(:+).duplicable?    # => false
```

`duplicable` coincide con el `dup` de Ruby según la versión de Ruby.

Así que en 2.4:

```ruby
nil.dup                 # => nil
:my_symbol.dup          # => :my_symbol
1.dup                   # => 1
 
nil.duplicable?         # => true
:my_symbol.duplicable?  # => true
1.duplicable?           # => true
```

Considerando que en 2.2 y 2.3:

```ruby
nil.dup                 # => TypeError: can't dup NilClass
:my_symbol.dup          # => TypeError: can't dup Symbol
1.dup                   # => TypeError: can't dup Fixnum
 
nil.duplicable?         # => false
:my_symbol.duplicable?  # => false
1.duplicable?           # => false
```

> Cualquier clase puede desautorizar la duplicación al eliminar `dup` y `clon` o levantar excepciones de ellos. Así, sólo el `rescue` puede decir si un objeto arbitrario dado es duplicable. `duplicable` Depende de la lista codificada anteriormente, pero es mucho más rápido que el `rescue`. Úselo sólo si sabe que la lista codificada es suficiente en su caso de uso.

### 2.4 deep\_dup

El método `deep_dup` devuelve una copia profunda de un objeto dado. Normalmente, cuando se hace `dup` a un objeto que contiene otros objetos, Ruby no los duplica, por lo que crea una copia poco profunda del objeto. Si tiene un array con una cadena, por ejemplo, se verá así:

```ruby
array     = ['string']
duplicate = array.dup
 
duplicate.push 'another-string'
 
# the object was duplicated, so the element was added only to the duplicate
array     # => ['string']
duplicate # => ['string', 'another-string']
 
duplicate.first.gsub!('string', 'foo')
 
# first element was not duplicated, it will be changed in both arrays
array     # => ['foo']
duplicate # => ['foo', 'another-string']
```

Como se puede ver, después de duplicar la instancia Array, tenemos otro objeto, por lo tanto podemos modificarlo y el objeto original permanecerá sin cambios. Sin embargo, esto no es cierto para los elementos del array. Puesto que el `dup` no hace la copia profunda, la secuencia dentro del arsenal sigue siendo el mismo objeto.

Si necesita una copia profunda de un objeto, debe utilizar `deep_dup`. Aquí hay un ejemplo:

```ruby
array     = ['string']
duplicate = array.deep_dup
 
duplicate.first.gsub!('string', 'foo')
 
array     # => ['string']
duplicate # => ['foo']
```

Si el objeto no es duplicable, `deep_dup` sólo lo devolverá:

```ruby
number = 1
duplicate = number.deep_dup
number.object_id == duplicate.object_id   # => true
```

### 2.5 try

Cuando desea llamar a un método en un objeto sólo si no es `nil`, la forma más sencilla de lograrlo es con sentencias condicionales, añadiendo desorden innecesario. La alternativa es usar `try`. `try` es como `Object#send` excepto que devuelve `nil` si se envía `nil`.

Aquí hay un ejemplo:

```ruby
# without try
unless @number.nil?
  @number.next
end
 
# with try
@number.try(:next)
```

Otro ejemplo es este código de `ActiveRecord::ConnectionAdapters::AbstractAdapter `donde `@logger` podría ser nil. Puede ver que el código utiliza `try` y evita una comprobación innecesaria.

```ruby
def log_info(sql, name, ms)
  if @logger.try(:debug?)
    name = '%s (%.1fms)' % [name || 'SQL', ms]
    @logger.debug(format_log_entry(name, sql.squeeze(' ')))
  end
end
```

PARAMOS EN ESTA PARTE DE ACTIVE SUPPORT POR LOS SIGUIENTES ASPECTOS:

* Esto es temario mas avanzado, donde voy a extender ciertas caracteristicas de rails
* A corto plazo necesitamos mas informacion sobre otros temas avanzados pero mas practicos, como por ejemplo Mailers, Background Jobs o Testing
* Mas adelante retomaremos la traduccion de estas Core Extensions





