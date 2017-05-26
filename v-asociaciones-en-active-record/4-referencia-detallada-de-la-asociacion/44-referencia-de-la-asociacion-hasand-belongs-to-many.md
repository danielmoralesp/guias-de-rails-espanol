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

#### 4.4.1.5 collection.destroy \(object, ...\) 

El método `collection.destroy` elimina uno o más objetos de la colección eliminando registros en la tabla de unión. Esto no destruye los objetos.

```ruby
@part.assemblies.destroy(@ assembly1)
```

#### 4.4.1.6 collection = \(objects\) 

El método `collection =` hace que la colección contenga sólo los objetos suministrados, agregando y eliminando según corresponda. Los cambios son persistentes en la base de datos

#### 4.4.1.7 collection\_singular\_ids 

El método `collection_singular_ids` devuelve una matriz de ids de los objetos de la colección.

```ruby
@assembly_ids = @part.assembly_ids
```

#### 4.4.1.8 collection\_singular\_ids = \(ids\) 

El método `collection_singular_ids =` hace que la colección contenga sólo los objetos identificados por los valores de clave primaria suministrados, añadiendo y eliminando según corresponda. Los son persistentes en la base de datos.

#### 4.4.1.9 collection.clear 

El método `collection.clear` elimina todos los objetos de la colección eliminando las filas de la tabla de unión. Esto no destruye los objetos asociados. 

#### 4.4.1.10 collection.empty? 

El método `collection.empty?` devuelve `true` si la colección no contiene ningún objeto asociado.

```ruby
<% If @ part.assemblies.empty? Unesdoc.unesco.org unesdoc.unesco.org
   Esta parte no se utiliza en ninguna asamblea
<% End%>
```

#### 4.4.1.11 collection.size 

El método `collection.size` devuelve el número de objetos de la colección.

```ruby
@assembly_count = @ part.assemblies.size
```

#### 4.4.1.12 collection.find \(...\) 

El método `collection.find` encuentra objetos dentro de la colección. Utiliza la misma sintaxis y opciones como `ActiveRecord :: Base.find`. También agrega la condición adicional de que el objeto debe estar en la colección.

```ruby
@assembly = @part.assemblies.find(1)
```

#### 4.4.1.13 collection.where \(...\) 

El método `collection.where` encuentra objetos dentro de la colección basados en las condiciones suministradas, pero los objetos se cargan perezosamente, lo que significa que la base de datos se consulta sólo cuando se accede a los objetos. También agrega la condición adicional de que el objeto debe estar en la colección.

```ruby
@new_assemblies = @part.assemblies.where("created_at>?", 2.days.ago)
```

#### 4.4.1.14 collection.exists? \(...\) 

El método `collection.exist?` comprueba si existe un objeto que cumple las condiciones suministradas en la colección. Utiliza la misma sintaxis y opciones como `ActiveRecord::Base.exists?`.

#### 4.4.1.15 collection.build\(attributes = {}\) 

El método `collection.buid` devuelve un nuevo objeto del tipo asociado. Este objeto se instanciará a partir de los atributos pasados y se creará el vínculo a través de la tabla de unión, pero el objeto asociado aún no se guardará.

```ruby
@assembly = @part.assemblies.build({assembly_name: "Transmission housing"})
```

#### 4.4.1.16 collection.create\(attributes = {}\) 

El método `collection.create` devuelve un nuevo objeto del tipo asociado. Este objeto se instanciará a partir de los atributos pasados, se creará el vínculo a través de la tabla de unión y, una vez que se pasen todas las validaciones especificadas en el modelo asociado, se guardará el objeto asociado.

```ruby
@assembly = @part.assemblies.create({assembly_name: "Transmission housing"})
```

#### 4.4.1.17 collection.create!\(attributes = {}\) 

Hace lo mismo que `collection.create`, pero dispara un `ActiveRecord::RecordInvalid` si el registro no es válido.



### 4.4.2 Opciones para has\_and\_belongs\_to\_many 

Mientras que Rails utiliza valores predeterminados inteligentes que funcionarán bien en la mayoría de las situaciones, puede haber ocasiones en las que desee personalizar el comportamiento de la referencia de asociación `has_and_belongs_to_many`. Tales personalizaciones pueden realizarse fácilmente pasando opciones al crear la asociación. Por ejemplo, esta asociación utiliza dos opciones:

```ruby
class Parts < ApplicationRecord
  has_and_belongs_to_many :assemblies, -> { readonly },
                                       autosave: true
end
```

La asociación `has_and_belongs_to_many` soporta estas opciones:

```ruby
:association_foreign_key
:autosave
:class_name
:foreign_key
:join_table
:validate
```

#### 4.4.2.1: association\_foreign\_key 

Por convención, Rails asume que la columna de la tabla de combinación utilizada para mantener la clave externa apuntando al otro modelo es el nombre de ese modelo con el sufijo `_id` añadido. La opción: `association_foreign_key` le permite establecer el nombre de la clave externa directamente: 

Las opciones: `foreign_key` y: `association_foreign_key` son útiles cuando se configura una self-join many-to-many. Por ejemplo:

```ruby
class User < ApplicationRecord
  has_and_belongs_to_many :friends,
      class_name: "User",
      foreign_key: "this_user_id",
      association_foreign_key: "other_user_id"
end
```

#### 4.4.2.2: autosave 

Si configura la opción: `autosave` en `true`, Rails guardará todos los miembros cargados y destruirá los miembros marcados para su destrucción cada vez que guarde el objeto principal.

#### 4.4.2.3: class\_name 

Si el nombre del otro modelo no puede derivarse del nombre de la asociación, puede utilizar la opción: `class_name` para proporcionar el nombre del modelo. Por ejemplo, si una parte tiene muchos ensamblajes, pero el nombre real del modelo que contiene los ensamblajes es `Gadget`, debería configurar las cosas de esta manera:

```ruby
class Parts < ApplicationRecord
  has_and_belongs_to_many :assemblies, class_name: "Gadget"
end
```

#### 4.4.2.4: foreign\_key 

Por convención, Rails asume que la columna de la tabla de combinación utilizada para mantener la clave externa apuntando a este modelo es el nombre de este modelo con el sufijo `_id` añadido. La opción: `foreign_key` le permite establecer el nombre de la clave externa directamente:

```ruby
class User < ApplicationRecord
  has_and_belongs_to_many :friends,
      class_name: "User",
      foreign_key: "this_user_id",
      association_foreign_key: "other_user_id"
end
```

#### 4.4.2.5: join\_table 

Si el nombre predeterminado de la tabla de combinación, basado en ordenamiento léxico, no es lo que desea, puede utilizar la opción: `join_table` para sustituir el valor predeterminado. 

#### 4.4.2.6: validate 

Si establece la opción: `validate` en `false`, los objetos asociados no se validarán cada vez que guarde este objeto. De forma predeterminada, esto es cierto: los objetos asociados se validarán cuando se guarde este objeto. 



### 4.4.3 Scope de has\_and\_belongs\_to\_many 

Puede haber ocasiones en las que desee personalizar la consulta utilizada por `has_and_belongs_to_many`. Tales personalizaciones se pueden conseguir a través de un bloque de scope. Por ejemplo:

```ruby
class Parts < ApplicationRecord
  has_and_belongs_to_many :assemblies, -> { where active: true }
end
```

Puede utilizar cualquiera de los métodos de consulta estándar dentro del bloque de ámbito. A continuación se analizan las siguientes:

```ruby
where
extending
group
includes
limit
offset
order
readonly
select
distinct
```

#### 4.4.3.1 where 

El método `where` permite especificar las condiciones que debe cumplir el objeto asociado.

```ruby
class Parts < ApplicationRecord
  has_and_belongs_to_many :assemblies,
    -> { where "factory = 'Seattle'" }
end
```

También puede establecer las condiciones mediante un hash:

```ruby
class Parts < ApplicationRecord
  has_and_belongs_to_many :assemblies,
    -> { where factory: 'Seattle' }
end
```

Si utiliza un estilo hash para el `where`, a continuación, la creación de registros a través de esta asociación se escaneará automáticamente utilizando el hash. En este caso, usar `@parts.assemblies.create` o `@parts.assemblies.build` creará órdenes donde la columna de fábrica tiene el valor "Seattle". 

#### 4.4.3.2 extending 

El método extendido especifica un módulo con nombre para extender el proxy de asociación. Las extensiones de la asociación se discuten en detalle más adelante en esta guía. 

#### 4.4.3.3 group 

El método `group` proporciona un nombre de atributo para agrupar el conjunto de resultados, utilizando una cláusula `GROUP BY` en el buscador `SQL`.

```ruby
class Parts < ApplicationRecord
  has_and_belongs_to_many :assemblies, -> { group "factory" }
end
```

#### 4.4.3.4 includes 

Puede utilizar el método `includes` para especificar asociaciones de segundo orden que deben cargarse con impaciencia cuando se utiliza esta asociación. 

#### 4.4.3.5 limit 

El método de límite le permite restringir el número total de objetos que se obtendrán a través de una asociación.

```ruby
class Parts < ApplicationRecord
  has_and_belongs_to_many :assemblies,
    -> { order("created_at DESC").limit(50) }
end
```

#### 4.4.3.6 offset 

El método `offset` le permite especificar el desplazamiento inicial para buscar objetos a través de una asociación. Por ejemplo, si ajusta el desplazamiento \(11\), omitirá los primeros 11 registros. 

#### 4.4.3.7 order 

El método `order` determina el orden en el que se recibirán los objetos asociados \(en la sintaxis utilizada por una cláusula `SQL ORDER BY`\).

```ruby
class Parts < ApplicationRecord
  has_and_belongs_to_many :assemblies,
    -> { order "assembly_name ASC" }
end
```

#### 4.4.3.8 readonly 

Si utiliza el método `readonly`, los objetos asociados serán de sólo lectura cuando se recuperen a través de la asociación. 

#### 4.4.3.9 select 

El método `select` le permite anular la cláusula `SQL SELECT` que se utiliza para recuperar datos sobre los objetos asociados. De forma predeterminada, Rails recupera todas las columnas. 

#### 4.4.3.10 distinct 

Utilice el método `distinct` para quitar duplicados de la colección. 



### 4.4.4 ¿Cuándo se guardan los objetos? 

Cuando asigna un objeto a una asociación `has_and_belongs_to_many`, ese objeto se guarda automáticamente \(para actualizar la tabla de unión\). Si asigna varios objetos en una sentencia, todos ellos se guardan.

Si alguno de estos registros falla debido a errores de validación, la instrucción de asignación devuelve `false` y la asignación en sí se cancela. 

Si el objeto principal \(el que declara la asociación `has_and_belongs_to_many`\) no se guarda \(es decir, `new_record?` Devuelve `true`\) los objetos secundarios no se guardan cuando se agregan. Todos los miembros no guardados de la asociación se guardarán automáticamente cuando se guarda el padre. 

Si desea asignar un objeto a una asociación `has_and_belongs_to_many` sin guardar el objeto, utilice el método `collection.build`.









