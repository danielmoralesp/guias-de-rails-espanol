# 9- Testeo de las Vistas

Testear la respuesta a su solicitud mediante la aserción de la presencia de elementos clave de HTML y su contenido es una forma común de probar las vistas de su aplicación. Al igual que las pruebas de ruta, las pruebas de vista residen en las `test/controllers/` o son parte de las pruebas del controlador. El método `assert_select` permite consultar elementos HTML de la respuesta mediante una sintaxis simple pero potente.

Hay dos formas de `assert_select`:

`assert_select(selector, [equality], [message])` asegura que la condición de igualdad se cumpla en los elementos seleccionados a través del selector. El selector puede ser una expresión de selector CSS \(String\) o una expresión con valores de sustitución.

`assert_select(element, selector, [equality], [message])` garantiza que la condición de igualdad se cumpla en todos los elementos seleccionados a través del selector a partir del elemento \(instancia de `Nokogiri::XML::Node` o `Nokogiri::XML::NodeSet`\) y sus descendientes.

Por ejemplo, puede verificar el contenido del elemento de título en su respuesta con:

```ruby
assert_select 'title', "Welcome to Rails Testing Guide"
```

También puede utilizar bloques `assert_select` anidados para una investigación más profunda.

En el ejemplo siguiente, el `assert_select` interno de `li.menu_item` se ejecuta dentro de la colección de elementos seleccionados por el bloque externo:

```ruby
assert_select 'ul.navigation' do
  assert_select 'li.menu_item'
end
```

Una colección de elementos seleccionados se puede iterar a través de ella de modo que `assert_select` puede ser llamado por separado en cada elemento.

Por ejemplo, si la respuesta contiene dos listas ordenadas, cada una con cuatro elementos de lista anidados, se pasarán las siguientes pruebas.

```ruby
assert_select "ol" do |elements|
  elements.each do |element|
    assert_select element, "li", 4
  end
end
 
assert_select "ol" do
  assert_select "li", 8
end
```

Esta aserción es muy poderosa. Para un uso más avanzado, consulte la [documentación](https://github.com/rails/rails-dom-testing/blob/master/lib/rails/dom/testing/assertions/selector_assertions.rb).

### 9.1 Aserciones adicionales basadas en la vista

Hay más afirmaciones que se utilizan principalmente en las vistas de prueba:

| Assertion | Purpose |
| :--- | :--- |
| `assert_select_email` | Le permite hacer aserciones en el cuerpo de un e-mail.. |
| `assert_select_encoded` | Le permite hacer aserciones sobre HTML codificado. Esto lo hace descodificando el contenido de cada elemento y luego llamando al bloque con todos los elementos no codificados. |
| `css_select(selector)`or`css_select(element, selector)` | Devuelve un array de todos los elementos seleccionados por el selector. En la segunda variante coincide primero con el elemento de base e intenta igualar esta expresión de selección a cualquiera de sus hijos. Si no hay coincidencias, ambas variantes devuelven un array vacío. |

He aquí un ejemplo de uso de `assert_select_email`:

```ruby
assert_select_email do
  assert_select 'small', 'Please click the "Unsubscribe" link if you want to opt-out.'
end
```



