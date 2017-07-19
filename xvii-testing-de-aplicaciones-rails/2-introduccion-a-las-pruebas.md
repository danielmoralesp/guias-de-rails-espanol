# 2- Introducción a las pruebas

El soporte para hacer testing en Rails se realizo desde el principio. No fue como una epifania tipo "oh, vamos a hacer el soporte para las pruebas de ejecución porque son nuevos y cool"

### 2.1 Rails se prepara para los testing desde la palabra Go

Rails crea un directorio de testing para usted tan pronto como crea un proyecto de Rails utilizando los `rails new application_name`. Si enlista el contenido de este directorio, verá:

```ruby
$ ls -F test
controllers/           helpers/               
mailers/               system/                                       test_helper.rb
fixtures/              integration/           
models/                application_system_test_case.rb
```

Los helpers, los mailers y los directorios de los modelos están destinados a realizar pruebas para los ayudantes de la vista, los mailers y los modelos, respectivamente. El directorio de controladores está destinado a realizar pruebas para controladores, rutas y vistas. El directorio de integración está destinado a realizar pruebas de interacciones entre controladores.

El directorio de prueba del sistema contiene pruebas del sistema, que se utilizan para la prueba completa del navegador de su aplicación. Las pruebas del sistema le permiten probar su aplicación de la misma manera que sus usuarios la experimentan y también ayudarle a probar su JavaScript. Las pruebas del sistema heredan de Capybara y realizan pruebas de navegador para su aplicación.

Los fixtures son una manera de organizar los datos de la prueba; Residen en el directorio `fixtures`.

También se creará un directorio de `jobs` cuando se genere una prueba asociada.

El archivo `test_helper.rb` contiene la configuración predeterminada para las pruebas.

El archivo `application_system_test_case.rb` contiene la configuración predeterminada para las pruebas del sistema.

