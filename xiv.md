# XIV - API de Internacionalización de Rails \(I18n\)

La gema Ruby I18n \(abreviatura de internacionalización\) que se entrega con Ruby on Rails \(a partir de Rails 2.2\) proporciona un framework fácil de usar y extensible para traducir su aplicación a un único idioma personalizado distinto al inglés o para proporcionar idiomas multilingües de soporte en su aplicación.

El proceso de "internacionalización" generalmente significa abstraer todas las cadenas y otros bits específicos de la configuración regional \(como los formatos de fecha o moneda\) de su aplicación. El proceso de "localización" significa proporcionar traducciones y formatos localizados para estos bits.

Por lo tanto, en el proceso de internacionalización de su aplicación de Rails usted tiene que:

* Asegúrese de que tiene soporte para i18n.
* Decirle a Rails dónde encontrar los diccionarios `locale`.
* Decirle a Rails cómo configurar, conservar y cambiar los entornos locales.

En el proceso de localización de su aplicación probablemente deseará hacer las siguientes tres cosas:

* Reemplazar o complementar la configuración regional predeterminada de Rails, como por ejemplo los formatos de fecha y hora, los nombres de mes, los nombres de modelos de Active Record, etc.
* Las cadenas abstractas en su aplicación dentro de diccionarios con clave , por ejemplo los mensajes flash, texto estático en sus vistas, etc.
* Guardar los diccionarios resultantes en alguna parte.

Esta guía lo guiará a través de la API I18n y contiene un tutorial sobre cómo internacionalizar una aplicación de Rails desde el principio.

Después de leer esta guía, sabrá:

* Cómo funciona I18n en Ruby on Rails
* Cómo utilizar correctamente I18n en una aplicación RESTful de varias maneras
* Cómo utilizar I18n para traducir los errores de Active Record o de Action Mailer en los subjects de correo electrónico
* Algunas otras herramientas para ir más allá con el proceso de traducción de su aplicación

> El framework Ruby I18n le proporciona todos los medios necesarios para la internacionalización/localización de su aplicación de Rails. También puedes usar varias gemas disponibles para agregar funciones o funcionalidades adicionales. Vea la [gema de rails-i18n](https://github.com/svenfuchs/rails-i18n) para más información.



