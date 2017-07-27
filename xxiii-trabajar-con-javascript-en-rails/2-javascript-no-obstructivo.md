# 2- JavaScript no intrusivo.

Rails utiliza una técnica llamada "JavaScript no intrusivo" para manejar la acción de adjuntar JavaScript al DOM. Esto generalmente se considera una mejor práctica dentro de la comunidad frontend, pero ocasionalmente puede leer tutoriales que muestran otras maneras de hacerlo.

Esta es la forma más sencilla de escribir JavaScript. Usted puede ver que esto se conoce como "JavaScript inline":

```html
<a href="#" onclick="this.style.backgroundColor='#990000'">Paint it red</a>
```

Cuando se hace clic, el fondo del enlace se volverá rojo. Aquí está el problema: ¿qué ocurre cuando tenemos un montón de JavaScript que queremos ejecutar en un clic?

```html
<a href="#" onclick="this.style.backgroundColor='#009900';this.style.color='#FFFFFF';">Paint it green</a>
```

Incómodo, ¿verdad? Podríamos extraer la definición de función del controlador de clics y convertirla en CoffeeScript:

```js
@paintIt = (element, backgroundColor, textColor) ->
  element.style.backgroundColor = backgroundColor
  if textColor?
    element.style.color = textColor
```

Y luego en nuestra página:

```html
<a href="#" onclick="paintIt(this, '#990000')">Paint it red</a>
```

Eso es un poco mejor, pero ¿qué pasa si tenemos múltiples enlaces que tienen el mismo efecto?

```html
<a href="#" onclick="paintIt(this, '#990000')">Paint it red</a>
<a href="#" onclick="paintIt(this, '#009900', '#FFFFFF')">Paint it green</a>
<a href="#" onclick="paintIt(this, '#000099', '#FFFFFF')">Paint it blue</a>
```

No muy DRY, ¿eh? Podemos arreglar esto usando los eventos en su lugar. Agregaremos un atributo `data-*` a nuestro enlace y, a continuación, enlazaremos un manejador al evento click de cada enlace que tenga ese atributo:

```js
@paintIt = (element, backgroundColor, textColor) ->
  element.style.backgroundColor = backgroundColor
  if textColor?
    element.style.color = textColor
 
$ ->
  $("a[data-background-color]").click (e) ->
    e.preventDefault()
 
    backgroundColor = $(this).data("background-color")
    textColor = $(this).data("text-color")
    paintIt(this, backgroundColor, textColor)
```

```html
<a href="#" data-background-color="#990000">Paint it red</a>
<a href="#" data-background-color="#009900" data-text-color="#FFFFFF">Paint it green</a>
<a href="#" data-background-color="#000099" data-text-color="#FFFFFF">Paint it blue</a>
```

Llamamos a este JavaScript 'no intrusivo' porque ya no estamos mezclando JavaScript en nuestro HTML. Hemos separado adecuadamente nuestras concerns, facilitando un cambio futuro. Podemos agregar fácilmente comportamiento a cualquier vínculo agregando el atributo `data`. Podemos ejecutar todo nuestro JavaScript a través de un minimizador y concatenador. Podemos servir todo nuestro paquete de JavaScript en cada página, lo que significa que se descargará en la carga de la primera página y luego se almacenará en caché en cada página después de eso. Un montón de pequeños beneficios que realmente se van sumando.

El equipo de Rails le anima a escribir su CoffeeScript \(y JavaScript\) en este estilo, y usted podrá esperar que muchas bibliotecas también sigan este patrón.



