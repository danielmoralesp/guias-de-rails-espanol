# 2- Convención de nombres del controlador

La convención de nombres de controladores en Rails favorece la pluralización de la última palabra en el nombre del controlador, aunque no es estrictamente necesario \(por ejemplo, `ApplicationController`\). Por ejemplo, `ClientsController` es preferible a `ClientController`, `SiteAdminsController` es preferible a `SiteAdminController` o `SitesAdminsController`, y así sucesivamente.

Siguiendo esta convención, le permitirá utilizar los routes generators \(o generadores de rutas\) predeterminados \(por ejemplo, `resources`, etc.\) sin necesidad de llamar cada `:path` o `:controller`, y mantendrá el uso de los path y los helpers de URL coherentes en toda la aplicación. Consulte la Guía de Layouts y Rendering para obtener más detalles.

> La convención de nombres del controlador difiere de la convención de nombres de los modelos, que se espera que sean nombrados en forma singular.



