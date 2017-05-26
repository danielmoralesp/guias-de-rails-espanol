# 4.2 Referencia de la asociación has\_one

La asociación `has_one` crea una coincidencia uno a uno con otro modelo. En términos de base de datos, esta asociación dice que la otra clase contiene la clave externa. Si esta clase contiene la clave externa, entonces debería usar `belongs_to` en su lugar.



### 4.2.1 Métodos agregados por has\_one

Cuando declara una asociación `has_one`, la clase declarante gana automáticamente cinco métodos relacionados con la asociación:

```ruby
association
association=(associate)
build_association(attributes = {})
create_association(attributes = {})
create_association!(attributes = {})
```

En todos estos métodos, la asociación se sustituye por el símbolo pasado como el primer argumento a `has_one`. Por ejemplo, dada la declaración:

```ruby
class Supplier < ApplicationRecord
  has_one :account
end
```

Cada instancia del modelo de proveedor tendrá estos métodos:

```ruby
Supplier.account
Supplier.account=
Supplier.build_account
Supplier.create_account
Supplier.create_account!
```

Al inicializar una nueva asociación `has_one` o `belongs_to`, debe utilizar el prefijo `build_` para crear la asociación, en lugar del método `association.build` que se utilizaría para las asociaciones `has_many` o `has_and_belongs_to_many`. Para crear uno, use el prefijo `create_`.

#### 4.2.1.1 association 

El método de asociación devuelve el objeto asociado, si lo hay. Si no se encuentra ningún objeto asociado, devuelve `nil`.

```ruby
@account = @supplier.account
```

Si el objeto asociado ya se ha recuperado de la base de datos para este objeto, se devolverá la versión almacenada en caché. Para anular este comportamiento \(y forzar una base de datos leída\), llame `#reload` en el objeto primario.

```ruby
@account = @supplier.reload.account
```

#### 4.2.1.2 association=\(associate\) 

El método `association=` asigna un objeto asociado a este objeto. Detrás de las escenas, esto significa extraer la clave primaria de este objeto y establecer la clave externa del objeto asociado al mismo valor.

```ruby
@supplier.account = @account
```

#### 4.2.1.3 build\_association\(attributes = {}\) 

El método `build_association` devuelve un nuevo objeto del tipo asociado. Este objeto será instanciado a partir de los atributos pasados, y el enlace a través de su clave externa se establecerá, pero el objeto asociado aún no se guardará.

```ruby
@account = @supplier.build_account(terms: "Net 30")
```

#### 4.2.1.4 create\_association\(attributes = {}\) 

El método `create_association` devuelve un nuevo objeto del tipo asociado. Este objeto se instanciará a partir de los atributos pasados, se establecerá el enlace a través de su clave externa y, una vez que se pasen todas las validaciones especificadas en el modelo asociado, se guardará el objeto asociado.

```ruby
@account = @supplier.create_account(terms: "Net 30")
```

#### 4.2.1.5 create\_association!\(attributes = {}\) 

Hace lo mismo que `create_association`, pero dispara `ActiveRecord::RecordInvalid` si el registro no es válido.



### 4.2.2 Opciones para has\_one

Mientras que Rails utiliza valores predeterminados inteligentes que funcionarán bien en la mayoría de las situaciones, puede haber ocasiones en las que desee personalizar el comportamiento de la referencia de asociación `has_one`. Tales personalizaciones pueden realizarse fácilmente pasando opciones al crear la asociación. Por ejemplo, esta asociación utiliza dos opciones:

```ruby
class Supplier < ApplicationRecord
  has_one :account, class_name: "Billing", dependent: :nullify
end
```

La asociación `has_one` soporta estas opciones:

```ruby
:as
:autosave
:class_name
:dependent
:foreign_key
:inverse_of
:primary_key
:source
:source_type
:through
:validate
```

#### 4.2.2.1 :as 

Establecer la opción: `as` indica que se trata de una asociación polimórfica. Las asociaciones polimórficas se discutieron en detalle anteriormente en esta guía. 

#### 4.2.2.2 :autosave 

Si configura la opción: `autosave` en `true`, Rails guardará todos los miembros cargados y destruirá los miembros marcados para su destrucción cada vez que guarde el objeto principal. 

#### 4.2.2.3 :class\_name 

Si el nombre del otro modelo no puede derivarse del nombre de la asociación, puede utilizar la opción: `class_name` para proporcionar el nombre del modelo. Por ejemplo, si un proveedor tiene una cuenta, pero el nombre real del modelo que contiene cuentas es Facturación, debería configurar las cosas de esta manera:

```ruby
class Supplier < ApplicationRecord
  has_one :account, class_name: "Billing"
end
```

#### 4.2.2.4 :dependent 

Controla qué sucede con el objeto asociado cuando se destruye su propietario:

* `:destroy` hace que el objeto asociado también sea destruido 
* `:delete` hace que el objeto asociado sea eliminado directamente de la base de datos \(para que las devoluciones de llamada no se ejecuten\) 
* `:nullify` hace que la clave externa se establezca en NULL. Las devoluciones de llamada no se ejecutan. 
* `:restrict_with_exception` hace que se eleve una excepción si hay un registro asociado 
* `:restrict_with_error` provoca que se agregue un error al propietario si hay un objeto asociado

Es necesario no establecer o dejar la opcion `:nullify` para aquellas asociaciones que tienen NULL en la base de datos con restricciones. Si no establece dependencia para destruir tales asociaciones, no podrá cambiar el objeto asociado porque la clave externa del objeto asociado inicial se establecerá en el valor `NULL` no permitido.

#### 4.2.2.5: foreign\_key 

Por convención, Rails asume que la columna utilizada para mantener la clave externa en el otro modelo es el nombre de este modelo con el sufijo `_id` añadido. La opción: `foreign_key` le permite establecer el nombre de la clave externa directamente:

```ruby
class Supplier < ApplicationRecord
  has_one :account, foreign_key: "supp_id"
end
```

En cualquier caso, Rails no creará columnas de clave externa para usted. Debe definirlos explícitamente como parte de sus migraciones.

#### 4.2.2.6 :inverse\_of 

La opción `:inverse_of` especifica el nombre de la asociación `belongs_to` que es la inversa de esta asociación. No funciona en combinación con las opciones `:through` o `:as`.

```ruby
class Supplier < ApplicationRecord
  has_one :account, inverse_of: :supplier
end
 
class Account < ApplicationRecord
  belongs_to :supplier, inverse_of: :account
end
```

#### 4.2.2.7: primary\_key 

Por convención, Rails asume que la columna utilizada para mantener la clave primaria de este modelo es `id`. Puede sobrescribirlo y especificar explícitamente la clave principal con la opción: `primary_key`.

#### 4.2.2.8: source 

La opción: `source` especifica el nombre de la asociación de origen para una asociación `has_one: through`.

#### 4.2.2.9: source\_type 

La opción: `source_type` especifica el tipo de asociación fuente para una asociación `has_one: through` que procede a través de una asociación polimórfica. 

#### 4.2.2.10: through 

La opción: `through` especifica un modelo de combinación mediante el cual realizar la consulta. Las asociaciones`has_one: through`  se discutieron en detalle anteriormente en esta guía. 

#### 4.2.2.11: validate 

Si establece la opción: `validate` en `true`, los objetos asociados se validarán cada vez que guarde este objeto. De forma predeterminada, esto es `false`: los objetos asociados no se validarán cuando se guarde este objeto.



### 4.2.3 Scope de has\_one 

Puede haber ocasiones en las que desee personalizar la consulta utilizada por `has_one`. Tales personalizaciones se pueden conseguir a través de un bloque de scope. Por ejemplo:

```ruby
class Supplier < ApplicationRecord
  has_one :account, -> { where active: true }
end
```

Puede utilizar cualquiera de los métodos de consulta estándar dentro del bloque de ámbito. A continuación se analizan las siguientes:

```ruby
where
includes
readonly
select
```

#### 4.2.3.1 where 

El método `where` permite especificar las condiciones que debe cumplir el objeto asociado.

```ruby
class Supplier < ApplicationRecord
  has_one :account, -> { where "confirmed = 1" }
end
```

#### 4.2.3.2 include 

Puede utilizar el método `include` para especificar asociaciones de segundo orden que deben cargarse con impaciencia cuando se utiliza esta asociación. Por ejemplo, considere estos modelos:

```ruby
class Supplier < ApplicationRecord
  has_one :account
end
 
class Account < ApplicationRecord
  belongs_to :supplier
  belongs_to :representative
end
 
class Representative < ApplicationRecord
  has_many :accounts
end
```

Si obtiene con frecuencia representantes directamente de los proveedores \(`@supplier.account.representative`\), puede hacer que su código sea un poco más eficiente al incluir representantes en la asociación de proveedores a cuentas:

```ruby
class Supplier < ApplicationRecord
  has_one :account, -> { includes :representative }
end

class Account < ApplicationRecord
  belongs_to :supplier
  belongs_to :representative
end
 
class Representative < ApplicationRecord
  has_many :accounts
end
```

#### 4.2.3.3 readonly 

Si utiliza el método `readonly`, el objeto asociado será de sólo lectura cuando se recupera a través de la asociación. 

#### 4.2.3.4 select 

El método `select` le permite anular la cláusula `SQL SELECT` que se utiliza para recuperar datos sobre el objeto asociado. De forma predeterminada, Rails recupera todas las columnas. 



### 4.2.4 ¿Existen objetos asociados? 

Puede ver si existen objetos asociados usando el método `association.nil?` :

```ruby
if @supplier.account.nil?
  @msg = "No account found for this supplier"
end
```



### 4.2.5 ¿Cuándo se guardan los objetos? 

Cuando asigna un objeto a una asociación `has_one`, ese objeto se guarda automáticamente \(para actualizar su clave externa\). Además, cualquier objeto que se está reemplazando también se guarda automáticamente, porque su clave externa también cambiará. 

Si cualquiera de estos guardados falla debido a errores de validación, la instrucción de asignación devuelve `false` y la asignación en sí se cancela. 

Si el objeto principal \(el que declara la asociación `has_one`\) no se guarda \(es decir, `new_record?` Devuelve `true`\) los objetos secundarios no se guardan. Automáticamente cuando se guarda el objeto principal.

Si desea asignar un objeto a una asociación `has_one` sin guardar el objeto, utilice el método `association.build`.

