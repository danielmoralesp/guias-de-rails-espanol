### 6. Parando la Ejecución

Como comienzas registrando nuevos callbaks para tus modelos, ellos serán encolados para su ejecución. Esta cola incluirá toda la validación de tu modelo, los callbaks registrados, y la operación en la base de datos que será ejecutada. 

La cadena entera de callbaks es envuelta en una transacción. Si algún método callback `before` retorna exactamente `false` o lanza una excepción, la cadena de ejecución se detiene y un hace `ROLLBACK`; los callbacks `after` solo pueden llevar esto a cabo lanzando una excepción. 

Alguna excepción que no es `ActiveRecord::Rollback` será relanzada por Rails después de la que la cadena de callbacks es detenida. Lanzando una excepción como `ActiveRecord::Rollback` puede romper el código de manera que no espere que métodos como `save` y `update_attributes` \(los cuales normalmente intentan retornar `true` o `false`\) para lanzar una excepción.



