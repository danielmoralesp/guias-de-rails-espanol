# 8- Generación de consultas inseguras

Debido a la forma en que Active Record interpreta los parámetros en combinación con la forma en que Rack analiza los parámetros de consulta, es posible emitir consultas de base de datos inesperadas con cláusulas `IS` `NULL`. Como respuesta a este problema de seguridad \(CVE-2012-2660, CVE-2012-2694 y CVE-2013-0155\) se introdujo el método `deep_munge` como una solución para mantener Rails seguro de forma predeterminada.

Un ejemplo de código vulnerable que podría ser utilizado por el atacante, si `deep_munge` no se realizó es:

```ruby
unless params[:token].nil?
  user = User.find_by_token(params[:token])
  user.reset_password!
end
```

Cuando `params[:token]` es uno de: `[nil], [nil, nil, ...]`  o `['foo', nil]` se omitirá la prueba de `nil`, pero `IS NULL` o `IN` \(`'foo', NULL` \) Donde las cláusulas todavía se agregarán a la consulta SQL.

Para mantener Rails seguro de forma predeterminada, `deep_munge` reemplaza algunos de los valores con `nil`. Debajo de la tabla se muestra lo que parecen los parámetros basados en JSON enviado en la solicitud:

| JSON | Parameters |
| :--- | :--- |
| `{ "person": null }` | `{ :person => nil }` |
| `{ "person": [] }` | `{ :person => [] }` |
| `{ "person": [null] }` | `{ :person => [] }` |
| `{ "person": [null, null, ...] }` | `{ :person => [] }` |
| `{ "person": ["foo", null] }` | `{ :person => ["foo"] }` |

Es posible volver a un comportamiento antiguo e inhabilitar `deep_munge` configurando su aplicación si usted es consciente del riesgo y sabe manejarlo:

```ruby
config.action_dispatch.perform_deep_munge = false
```



