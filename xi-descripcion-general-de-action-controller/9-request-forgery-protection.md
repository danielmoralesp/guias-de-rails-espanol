# 9- Request Forgery Protection

La falsificación de solicitudes entre sitios es un tipo de ataque en el que un sitio engaña a un usuario para que realice solicitudes en otro sitio, posiblemente agregando, modificando o eliminando datos en ese sitio sin el conocimiento o permiso del usuario.

El primer paso para evitar esto es asegurarse de que todas las acciones "destructivas" \(`create`, `update` y `destroy`\) sólo se puede acceder con las solicitudes no-GET. Si sigues las convenciones RESTful ya lo estás haciendo. Sin embargo, un sitio malicioso todavía puede enviar una solicitud no GET a su sitio con bastante facilidad, y ahí es donde entra en juego la protección contra la falsificación de solicitudes. Como dice el nombre, protege de las solicitudes falsificadas.

La forma en que se hace esto es agregando un token no adivinable que sólo conoce su servidor en cada solicitud. De esta manera, si una solicitud llega sin el token adecuado, se le denegará el acceso.

Si genera un formulario como este:

```ruby
<%= form_for @user do |f| %>
  <%= f.text_field :username %>
  <%= f.text_field :password %>
<% end %>
```

Verá cómo se agrega el token como un campo oculto:

```html
<form accept-charset="UTF-8" action="/users/1" method="post">
<input type="hidden"
       value="67250ab105eb5ad10851c00a5621854a23af5489"
       name="authenticity_token"/>
<!-- fields -->
</form>
```

Rails agrega este token a cada formulario que se genera utilizando los ayudantes de formulario, por lo que la mayor parte del tiempo no tiene que preocuparse por ello. Si está escribiendo un formulario manualmente o necesita agregar el token por otro motivo, está disponible a través del método `form_authenticity_token`:

`form_authenticity_token` genera un token de autenticación válido. Esto es útil en lugares en los que Rails no lo agrega automáticamente, como en las llamadas personalizadas de Ajax.

La Guía de seguridad tiene más información sobre esto y muchos otros problemas relacionados con la seguridad que debe tener en cuenta al desarrollar una aplicación web.

