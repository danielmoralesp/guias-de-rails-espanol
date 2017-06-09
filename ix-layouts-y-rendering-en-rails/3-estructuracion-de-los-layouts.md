# 3 Estructuración de los layouts

Cuando Rails renderiza una vista como una respuesta, lo hace combinando la vista con el layout actual, utilizando las reglas para encontrar el layout actual que se trató anteriormente en esta guía. Dentro de un layout, tiene acceso a tres herramientas para combinar diferentes bits de salida para formar la respuesta global:

* Asset tags helpers
* `yield` y `content_for`
* Partials



