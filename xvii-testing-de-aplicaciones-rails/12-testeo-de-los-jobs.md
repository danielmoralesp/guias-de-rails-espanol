# 12- Testeo de los Jobs

Dado que sus jobs personalizados pueden estar en cola en diferentes niveles dentro de su aplicación, tendrá que probar ambos, los jobs mismos \(su comportamiento cuando se ponen en cola\) y que otras entidades los encolen correctamente.

### 12.1 Un caso de prueba básico

De forma predeterminada, al generar un job, una prueba asociada se generará también en el directorio `test/jobs`. Aquí hay una prueba de ejemplo con un jobs de facturación:

```ruby
require 'test_helper'
 
class BillingJobTest < ActiveJob::TestCase
  test 'that account is charged' do
    BillingJob.perform_now(account, product)
    assert account.reload.charged_for?(product)
  end
end
```

Esta prueba es bastante simple y sólo afirma que el job obtenga el trabajo realizado como se esperaba.

De forma predeterminada, `ActiveJob::TestCase` establecerá el adaptador de cola en `:test` para que sus trabajos se realicen en línea. También se asegurará de que todos los trabajos ejecutados previamente y en cola se borren antes de cualquier ejecución de prueba para que pueda asumir con seguridad que ya no se han ejecutado trabajos en el ámbito de cada prueba.

### 12.2 Aserciones personalizadas y pruebas de jobs dentro de otros componentes

Active Job envía con un montón de aserciones personalizadas que se pueden utilizar para disminuir la verbosidad de las pruebas. Para obtener una lista completa de aserciones disponibles, consulte la documentación de API para `ActiveJob::TestHelper`.

Es una buena práctica asegurarse de que sus jobs se pongan en cola correctamente o se realicen dondequiera que los invoque \(por ejemplo, dentro de sus controladores\). Esto es precisamente donde las aserciones personalizadas proporcionadas por Active Job son bastante útiles. Por ejemplo, dentro de un modelo:

```ruby
require 'test_helper'
 
class ProductTest < ActiveJob::TestCase
  test 'billing job scheduling' do
    assert_enqueued_with(job: BillingJob) do
      product.charge(account)
    end
  end
end
```



