# 1 ¿Qué es una aplicación API?

Tradicionalmente, cuando la gente decía que utilizaban Rails como "API", significaba proporcionar una API accesible mediante programación junto con su aplicación web. Por ejemplo, GitHub proporciona una API que puede utilizar desde sus propios clientes personalizados.

Con la llegada de los frameworks client-side, más desarrolladores están utilizando Rails para construir un back-end que se comparte entre su aplicación web y otras aplicaciones nativas.

Por ejemplo, Twitter utiliza su API pública en su aplicación web, que se construye como un sitio estático que consume recursos JSON.

En lugar de usar Rails para generar HTML que se comunica con el servidor a través de formularios y enlaces, muchos desarrolladores están tratando su aplicación web como un cliente API entregado como HTML con JavaScript que consume una API JSON.

Esta guía cubre la creación de una aplicación de Rails que sirve recursos JSON a un cliente de API, incluidos los frameworks del lado del cliente.



