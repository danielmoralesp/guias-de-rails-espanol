# 4- Pruebas de modelos

Las pruebas del modelo se utilizan para probar varios modelos de su aplicación.

Las pruebas del modelo Rails se almacenan en el directorio `test/models`. Rails proporciona un generador para crear un esqueleto de pruebas de modelo por usted.  

```ruby
$ bin/rails generate test_unit:model article title:string body:text
create  test/models/article_test.rb
create  test/fixtures/articles.yml
```

Las pruebas de modelo no tienen su propia superclase como `ActionMailer::TestCase` en cambio heredan de `ActiveSupport::TestCase`.



