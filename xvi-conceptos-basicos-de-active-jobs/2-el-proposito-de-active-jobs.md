# 2- El Propósito de Active Jobs

El punto principal es asegurar que todas las aplicaciones de Rails tendrán la infraestructura de jobs en su lugar. Podemos entonces tener las características del framework y otras gemas construidas sobre eso, sin tener que preocuparse por las diferencias de las APIs entre varios corredores de jobs como Delayed Job y Resque. Entonces, escoger el backend de cola se convierte en una preocupación operacional. Y usted podrá variar entre ellos sin tener que reescribir sus jobs.

> Rails viene de forma predeterminada con una implementación de cola asíncrona que ejecuta trabajos con un grupo de subprocesos en proceso. Los trabajos se ejecutarán de forma asíncrona, pero cualquier trabajo de la cola se eliminará al reiniciar.



