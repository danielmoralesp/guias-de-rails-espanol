# 7. Active Record y la Integridad Referencial

La metodología de Active Record dice que la inteligencia pertenece al modelo, no a la base de datos. Así bien, características tales como disparadores o restricciones, las cuales otorgan algo de inteligencia a la base de datos, no son fuertemente utilizadas. 

Las validaciones tal como `validates :foreign_key, uniqueness: true` son un modo por el cual los modelos pueden forzar la integridad de los datos. La opción `:dependent` en las asociaciones permite a los modelos destruir los objetos que son hijos cuando el padre es destruido. Como cualquier cosa que opere a nivel de aplicación, eso no garantiza integridad referencial, entonces algunas personas se aseguran esta con restricciones sobre las claves foráneas en la base de datos. 

Sin embargo Active Record no provee todas las herramientas para trabajar directamente con algunas características, el método `execute` puede ser utilizado para ejecutar un SQL arbitrario.



