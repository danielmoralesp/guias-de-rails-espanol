# 1- Descripción General: Cómo encajan las piezas

Esta guía se centra en la interacción entre Controlador y Vista en el triángulo Modelo-Vista-Controlador. Como saben, el Controlador es responsable de orquestar todo el proceso de manejo de una solicitud en Rails, aunque normalmente entrega cualquier código pasado al Modelo. Pero entonces, cuando es hora de enviar una respuesta de vuelta al usuario, el controlador entrega las cosas a la vista. Esa entrega es el tema de esta guía.

En trazos generales, esto implica decidir qué se debe enviar como respuesta y llamar a un método apropiado para crear esa respuesta. Si la respuesta es una vista completa, Rails también hace un trabajo extra para envolver la vista en un layout y posiblemente tambien deba obtener vistas parciales, y hace ese trabajo. Verás todas esas rutas más adelante en esta guía.

