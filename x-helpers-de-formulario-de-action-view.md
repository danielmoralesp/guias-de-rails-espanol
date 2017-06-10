# X- Helpers de Formulario de Action View

Los formularios en las aplicaciones web son una interfaz esencial para la entrada de datos por parte de los usuarios. Sin embargo, las etiquetas de los formularios pueden convertirse rápidamente en un trabajo tedioso de escribir y de mantener debido a la necesidad de manejar diferentes nombres para el control del formulario y sus numerosos atributos. Rails elimina esta complejidad proporcionando helpers de vista para generar las etiquetas de los formularios. Sin embargo, dado que estos helpers tienen diferentes casos de uso, los desarrolladores necesitan conocer las diferencias entre los métodos de ayuda antes de utilizarlos.

Después de leer esta guía, sabrá:

* Cómo crear formularios de búsqueda y otros formularios genéricos similares, que no representan ningún modelo específico en su aplicación.
* Cómo crear formularios orientados a los modelos para crear y editar registros de base de datos específicos.
* Cómo generar cuadros de selección \(select box\) de varios tipos de datos.
* Qué helpers de fecha y hora ofrece Rails.
* Como hacer un formulario que suba archivos.
* Cómo publicar formularios en recursos externos y especificar la configuración de un `authenticity_token`.
* Cómo construir formularios complejos.

> Esta guía no pretende ser una documentación completa de los helpers de formulario disponibles y de sus argumentos. Visite la documentación de la API de Rails para obtener una referencia completa.



