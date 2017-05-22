# 1. ¿Que es Active Record?

Active Record es la M en MVC - el modelo - el cual es la capa del sistema responsable de representar los datos y la lógica de negocio para manipularlos. Active Record facilita la creación y manipulación de objetos de negocio quienes requieren ser almacenados persistentemente en una base de datos. Esta es una implementación del patrón de Active Record el cual en si mismo es una descripción de un sistema de Mapeo de Objetos Relacionales.

### 1.1 El Patrón Active Record 

Active Record fue descrito por Martin Fowler en su libro Patterns of Enterprise Application Architecture. En Active Record, los objetos soportan tanto la persistencia y el comportamiento que opera con los datos. Active Record toma la opinion que al asegurar el acceso lógico a los datos como parte del objeto educará a los usuarios de ese objeto sobre como escribir y leer en la base de datos.

### 1.2 Mapeo de Objetos Relacionales 

El Mapeo de Objetos Relacionales \(Object-Relational Mapping\), comunmente nombrado por sus siglas ORM, es una técnica que conecta la riqueza de los objetos de una aplicación con las tablas de un sistema de base de datos relacional. Utilizando ORM, las propiedades y las relaciones de los objetos en una aplicación pueden ser fácilmente guardadas y recuperadas desde la base de datos sin escribir sentencias SQL directamente y con el mínimo código en general de acceso a la base de datos.

### 1.3 Active Record como un Framework ORM 

Active Record brinda varios mecanismos, los más importantes nos dan la capacidad para: 

* Representar modelos y sus datos. 
* Representar asociaciones entre esos modelos. 
* Representar jerarquías de herencia a través de modelos relacionados. 
* Validar modelos antes de que sean guardados o cambiados en la base de datos. 
* Mantener las operaciones de la base de datos orientadas a objetos.



