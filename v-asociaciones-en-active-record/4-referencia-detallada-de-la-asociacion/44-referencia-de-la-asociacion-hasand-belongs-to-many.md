# 4.4 Referencia de la asociación has\_and\_belongs\_to\_many

La asociación `has_and_belongs_to_many` crea una relación many-to-many con otro modelo. En términos de base de datos, esto asocia dos clases a través de una tabla de unión intermedia que incluye claves externas que se refieren a cada una de las clases. 



### 4.4.1 Métodos Añadido por has\_and\_belongs\_to\_many

Cuando declara una asociación `has_and_belongs_to_many`, la clase declarante obtiene automáticamente 16 métodos relacionados con la asociación:

```ruby
collection
collection<<(object, ...)
collection.delete(object, ...)
collection.destroy(object, ...)
collection=(objects)
collection_singular_ids
collection_singular_ids=(ids)
collection.clear
collection.empty?
collection.size
collection.find(...)
collection.where(...)
collection.exists?(...)
collection.build(attributes = {})
collection.create(attributes = {})
collection.create!(attributes = {})
```

En todos estos métodos, la colección se sustituye por el símbolo pasado como el primer argumento a `has_and_belongs_to_many`, y `collection_singular` es reemplazado por la versión singularizada de ese símbolo. Por ejemplo, dada la declaración:

```ruby
class Part < ApplicationRecord
  has_and_belongs_to_many :assemblies
end
```

Cada instancia del modelo de `Part` tendrá estos métodos:

```ruby
Part.assemblies
Part.assemblies<<(object, ...)
Part.assemblies.delete(object, ...)
Part.assemblies.destroy(object, ...)
Part.assemblies=(objects)
Part.assembly_ids
Part.assembly_ids=(ids)
Part.assemblies.clear
Part.assemblies.empty?
Part.assemblies.size
Part.assemblies.find(...)
Part.assemblies.where(...)
Part.assemblies.exists?(...)
Part.assemblies.build(attributes = {}, ...)
Part.assemblies.create(attributes = {})
Part.assemblies.create!(attributes = {})
```

#### 4.4.1.1 Métodos de columna adicionales 

Si la tabla de unión de una asociación `has_and_belongs_to_many` tiene columnas adicionales más allá de las dos claves foráneas, estas columnas se agregarán como atributos a los registros recuperados a través de esa asociación. Los registros devueltos con atributos adicionales siempre serán de sólo lectura, ya que Rails no puede guardar los cambios en esos atributos. 

El uso de atributos adicionales en la tabla de combinación en una asociación `has_and_belongs_to_many` está obsoleto. Si requiere este tipo de comportamiento complejo en la tabla que une dos modelos en una relación muchos-a-muchos, debería usar una asociación `has_many: through` en lugar de `has_and_belongs_to_many`. 

#### 4.4.1.2 collection 

El método `collection` devuelve una matriz de todos los objetos asociados. Si no hay objetos asociados, devuelve una matriz vacía.

```ruby
@assemblies = @part.assemblies
```

#### 4.4.1.3 collection&lt;&lt;\(object, ...\) 

El método `collection <<` agrega uno o más objetos a la colección creando registros en la tabla de unión. Este método se denomina `collection.concat` y `collection.push`.

#### 4.4.1.4 collection.delete \(object, ...\) 

El método `collection.delete` elimina uno o más objetos de la colección eliminando registros en la tabla de unión. Esto no destruye los objetos.

```ruby
@part.assemblies.delete(@assembly1)
```









