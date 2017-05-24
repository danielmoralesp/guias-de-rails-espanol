# 6. Volcando el Esquema y Tú

### 6.1 ¿Para qué son los Ficheros de Esquema? 

Las migraciones, con lo poderosas que pueden ser, no son la fuente fiel de tu esquema de base de datos. Este rol cae sobre ya sea sobre el fichero `db/schema.rb` o un fichero SQL generado por Active Record al examinar la base de datos. Estos no están diseñados para ser editados, representan solo el estado actual del esquema de la base de datos.

No es necesario \(y eso es un error repetido\) desplegar una nueva instancia de una aplicación al volver a ejecutar todo el historial de migraciones. Es mucho más simple y rápido cargar en la base de datos la descripción del esquema actual.

Por ejemplo, así es como la base de datos de `test` es creada: la actual base de datos de desarrollo es volcada \(ya sea desde `db/schema.rb` o desde `db/structure.sql`\) y luego grabada en la base de datos de `test`. 

El fichero de esquema también es útil si quieres echar un vistazo a los atributos que un objeto Active Record tiene. Esta información no está en el código de los modelos y es frecuentemente propagada a través de varias migraciones, pero esta información está muy bien resumida en el fichero de esquema. 

La gema `annotate_models` \(https://github.com/ctran/annotate\_models\) automáticamente añade y actualiza comentarios en la parte superior de cada modelo resumiendo el esquema si deseas esa funcionalidad.



### 6.2 Tipos de Volcado del Esquema

Hay 2 maneras de volcar el esquema. Esto se configura en `config/application.rb` a través de la directiva `config.active_record.schema_format`, la cual puede ser o bien `:sql` o `:ruby`. Si está seleccionado `:ruby` entonces el esquema es guardado en `db/schema.rb`. Si miras este fichero encontrarás que luce casi como una gran migración:

```ruby
ActiveRecord::Schema.define(version: 20080906171750) do
  create_table "authors", force: true do |t|
    t.string   "name"
    t.datetime "created_at"
    t.datetime "updated_at"
  end
 
  create_table "products", force: true do |t|
    t.string   "name"
    t.text "description"
    t.datetime "created_at"
    t.datetime "updated_at"
    t.string "part_number"
  end
end
```

En muchos sentidos esto es exactamente lo que es. Este fichero fue creado inspeccionando la base de datos y expresando su estructura utilizando `create_table`, `add_index`, etc. Porque es independiente de la base de datos, podría ser cargado en cualquier base de datos que Active Record soporte. Esto podría ser muy útil si tienes que distribuir una aplicación que debe estar disponible para ejecutarse contra multiples bases de datos. 

Sin embargo hay algunas funcionalidades que quedan fuera: `db/schema.rb` no puede expresar items específicos de una base de datos como disparadores, o procedimientos almacenados. Mientras que en una migración tu puedes ejecutar definiciones SQL personlizadas, el volcado de esquema no puede reconstituir aquellas definiciones desde la base de datos. Si estás utilizando características como estas, entonces deberías configuar el formato del esquema a `:sql`. 

En lugar de utilizar el volcado del esquema de Active Record, la estructura de la base de datos será volcada utilizando una herramienta específica de la base de datos dentro del `db/structure.sql` \(via la tarea Rake `db:structure:dump`\). Por ejemplo, para PostgreSQL, se utiliza `pg_dump`. Para MySQL, este fichero contendrá la salida `SHOW CREATE TABL`de te varias tablas. 

Cargar estos esquemas es sencillamente cuestión de ejecutar las definiciones SQL que contienen. Por definción, esto creará una copia perfecta de la estructura de la base de datos. Utilizando el formato del esquema `:sql` sin embargo, vamos a asegurarnos la carga del esquema en otro RDBMS\(Relational Data Base Manager System -  Sistema de gestión de bases de datos relacionales \) distinto al que fue utilizado para crearlo.



### 6.3 Volcados del Esquema y Control de la Fuente

Los volcados del esquema son el código fuente que manda a tu esquema de base de datos.

`db/schema.rb` contiene la versión actual de la base de datos. Esto asegura que ocurrirán conflictos en el caso de que se mezclen dos ramas que han tocado el esquema. Cuando esto ocurre, resuelve los conflictos manualmente, manteniendo el número de versión más alto entre los dos.





