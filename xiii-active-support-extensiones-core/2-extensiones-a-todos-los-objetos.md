# 2- Extensiones a todos los objetos

### 2.1 `blank?` Y `present?`

Los valores siguientes se consideran en blanco en una aplicación de Rails:

* `nil` y `false`,
* Strings compuestos sólo de espacios en blanco \(ver nota más abajo\),
* Arrays h hashes vacíos
* Cualquier otro objeto que responde a `empty?` Y está vacío.

> El predicado de las cadenas usa los caracteres Unicode-aware para la `class[:space:]`, por lo que por ejemplo U+2029 \(separador de párrafos\) se considera como espacio en blanco.

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

Se define en `active_support/core_ext/object/blank.rb`.



