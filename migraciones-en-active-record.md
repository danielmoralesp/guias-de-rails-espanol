# Migraciones Active Record

Las migraciones son la característica de Active Record que permite mantener tu esquema de base de datos a través del tiempo. En lugar de escribir las modificaciones del esquema en SQL puro, las migraciones te permiten utilizar un lenguaje de definición fácil en Ruby DSL, para describir cambios para tus tablas. Después de leer esta guía sabrás: 

* Los comandos generadores que puedes utilizar para crearlas. 
* Los métodos que Active Record provee para manipular tu base de datos. 
* Las tareas Rake para manipular las migraciones y el esquema. 
* Cómo las migraciones afectan al fichero schema.rb.

### Schema Migration

No debe confundirse con la migración de datos. En la ingeniería de software, la migración de esquemas \(también migración de base de datos, gestión de cambios de base de datos\) se refiere a la gestión incremental, cambios reversibles a los esquemas de base de datos relacional. Se realiza una migración de esquema en una base de datos siempre que sea necesario actualizar o revertir el esquema de dicha base de datos a alguna versión más reciente o anterior. 

Las migraciones se realizan mediante programación utilizando una herramienta de migración de esquema. Cuando se invoca con una versión de esquema deseada especificada, la herramienta automatiza la aplicación sucesiva o inversión de una secuencia apropiada de cambios de esquema hasta que se lleva al estado deseado. 

La mayoría de las herramientas de migración de esquemas tienen como objetivo minimizar el impacto de los cambios de esquema en cualquier dato existente en la base de datos. A pesar de esto, la preservación de datos en general no está garantizada porque los cambios de esquema como la eliminación de una columna de base de datos pueden destruir datos \(es decir, todos los valores almacenados en esa columna para todas las filas de esa tabla se eliminan\). En su lugar, las herramientas ayudan a preservar el significado de los datos o a reorganizar los datos existentes para satisfacer nuevos requisitos. Dado que el significado de los datos a menudo no puede codificarse, la configuración de las herramientas suele necesitar intervención manual. 

#### Riesgos y beneficios

La migración de esquemas permite corregir errores y adaptar los datos a medida que cambian los requisitos. Son una parte esencial de la evolución del software, especialmente en entornos ágiles \(ver más abajo\).

La aplicación de una migración de esquema a una base de datos de producción siempre es un riesgo. Las bases de datos de desarrollo y pruebas tienden a ser más pequeñas y limpias. Los datos en ellos se entienden mejor o, si todo falla, la cantidad de datos es lo suficientemente pequeña como para que un humano pueda procesar. Las bases de datos de producción son generalmente enormes, viejas y llenas de sorpresas. Las sorpresas pueden provenir de muchas fuentes:

* Los datos dañados que fueron escritos por las versiones viejas del software y que no fueron limpiadas correctamente 
* Dependencias implícitas en los datos que nadie más conoce
* Personas que cambian directamente la base de datos sin utilizar las herramientas designadas 
* Errores en las herramientas de migración del esquema 
* Errores en los supuestos de cómo se deben migrar los datos

Por estas razones, el proceso de migración necesita un alto nivel de disciplina, pruebas exhaustivas y una estrategia de copia de seguridad.



