# 4. Seleccionando campos específicos

De forma predeterminada, `Model.find` selecciona todos los campos del conjunto de resultados mediante `select *`.

Para seleccionar sólo un subconjunto de campos del conjunto de resultados, puede especificar el subconjunto mediante el método `select`.

Por ejemplo, para seleccionar sólo las columnas `viewable_by` y `locked`:

```ruby
Client.select("viewable_by, locked")
```

La consulta `SQL` utilizada por esta llamada de búsqueda sería algo como:

```ruby
SELECT viewable_by, locked FROM clients
```

Tenga cuidado porque esto también significa que está inicializando un objeto de modelo con sólo los campos que ha seleccionado. Si intenta acceder a un campo que no está en el registro inicializado, recibirá:

```ruby
ActiveModel::MissingAttributeError: missing attribute: <attribute>
```

Donde `<attribute>` es el atributo que solicitó. El método `id` no disparará el `ActiveRecord::MissingAttributeError,` así que tenga cuidado al trabajar con asociaciones porque necesitan el método `id` para funcionar correctamente.

Si desea capturar sólo un registro por valor único en un campo determinado, puede utilizar `distinct`:

```ruby
Client.select(:name).distinct
```

Esto generaría un `SQL` como:

```ruby
SELECT DISTINCT name FROM clients
```

También puede eliminar la restricción de unicidad:

```ruby
query = Client.select(:name).distinct
# => Retorna names únicos
 
query.distinct(false)
# => Returns todos los names, incluso si están duplicados
```



