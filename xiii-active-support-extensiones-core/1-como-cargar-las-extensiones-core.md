# 1- Cómo cargar las extensiones core

### 1.1 Active Support autónomo

Active Support no carga nada de forma predeterminada. Se divide en pedazos pequeños de modo que usted pueda cargar lo que necesita, y también tiene algunos entry points de conveniencia para cargar las extensiones relacionadas en un solo shot, incluso todo.

Lo hacemos, después de un simple `require` como:

```ruby
require 'active_support'
```

sus objetos no responden a `blank?`. Veamos cómo cargar su definición.

#### 1.1.1 Selección de lo que desea en una sola definición

La manera más ligera de obtener `blank?` Es escoger el archivo que lo define.

Para cada método definido como una extensión del núcleo \(core\), esta guía tiene una nota que indica dónde se define dicho método. En el caso de `blank?` La nota dice:

Se define en `active_support/core_ext/object/blank.rb.`

Eso significa que puedes requerirlo así:

```ruby
require 'active_support'
require 'active_support/core_ext/object/blank'
```

Active Support ha sido cuidadosamente revisado para que la selección unitaria de un archivo, cargue sólo estrictamente lo necesario y las dependencias, si las hay.

#### 1.1.2 Carga de extensiones de núcleo agrupadas

El siguiente paso es simplemente cargar todas las extensiones del Object. Como regla general, las extensiones de SomeClass están disponibles en una linea cargando: `active_support/core_ext/some_class`

Por lo tanto, para cargar todas las extensiones de objeto \(incluyendo `blank?`\):

```ruby
require 'active_support'
require 'active_support/core_ext/object'
```

#### 1.1.3 Cargando todas las extensiones de núcleo

Es posible que prefiera cargar todas las extensiones principales, hay un archivo para eso:

```ruby
require 'active_support'
require 'active_support/core_ext'
```

#### 1.1.4 Cargando todo el active support

Y, por último, si quieres tener disponible todo el active support disponible:

```ruby
require 'active_support/all'
```

Que ni siquiera es necesario poner todo el active support en la memoria por adelantado, de hecho algunas cosas se configuran a través de la carga automática, por lo que sólo se carga si se utiliza.

### 1.2 Active Support dentro de una aplicación Ruby on Rails

Una aplicación de Ruby on Rails carga todo el Soporte activo a menos que `config.active_support.bare` sea verdadero. En ese caso, la aplicación sólo cargará lo que el propio framework selecciona para sus propias necesidades, y aún puede seleccionarse a sí misma en cualquier nivel de granularidad, como se explicó en la sección anterior.

