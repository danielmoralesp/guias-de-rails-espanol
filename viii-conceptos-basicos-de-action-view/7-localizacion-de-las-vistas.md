# 7- Vistas con Locale

Action View tiene la capacidad de procesar diferentes plantillas dependiendo de la configuración local actual. 

Por ejemplo, supongamos que tiene un `ArticlesController` con una acción `show`. De forma predeterminada, al llamar a esta acción, renderizamos `app/views/articles/show.html.erb`. Pero si usted fija `I18n.locale =: de`, a continuación, `app/views/articles/show.de.html.erb` se renderizará en su lugar. Si la plantilla localizada no está presente, se usará la versión no decorada. Esto significa que no está obligado a proporcionar vistas localizadas para todos los casos, pero se prefieren y utilizan si están disponibles.

Puede utilizar la misma técnica para localizar los archivos de rescate en su directorio público. Por ejemplo, establecer `I18n.locale =: de` y crear `public/500.de.html` y `public/404.de.html` le permitiría tener páginas de rescate localizadas.

Dado que Rails no restringe los símbolos que utiliza para configurar I18n.locale, puede aprovechar este sistema para mostrar contenido diferente dependiendo de lo que quiera. Por ejemplo, suponga que tiene algunos usuarios "expertos" que deberían ver diferentes páginas en comparación a los usuarios "normales". Podría agregar lo siguiente a `app/controllers/application.rb`:

```ruby
before_action :set_expert_locale
 
def set_expert_locale
  I18n.locale = :expert if current_user.expert?
end
```

A continuación, puede crear vistas especiales como `app/views/articles/show.expert.html.erb` que sólo se mostrará a los usuarios expertos.

