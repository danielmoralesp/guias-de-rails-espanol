# 8- Formularios a Recursos Externos

Los helpers de formulario de Rails también se pueden usar para crear un formulario y publicar datos en un recurso externo. Sin embargo, a veces puede ser necesario establecer un `authenticity_token` para el recurso; Esto puede hacerse pasando un parámetro `authenticity_token: 'your_external_token'` a las opciones `form_tag`:

```ruby
<%= form_tag 'http://farfar.away/form', authenticity_token: 'external_token' do %>
  Form contents
<% end %>
```

A veces, cuando se envían datos a un recurso externo, como una pasarela de pago, los campos que se pueden utilizar en el formulario están limitados por una API externa y puede ser indeseable generar un `authenticity_token`. Para no enviar un `token`, simplemente pase `false` a la opción `:authenticity_token`:

```ruby
<%= form_tag 'http://farfar.away/form', authenticity_token: false do %>
  Form contents
<% end %>
```

La misma técnica también está disponible para `form_for`:

```ruby
<%= form_for @invoice, url: external_url, authenticity_token: 'external_token' do |f| %>
  Form contents
<% end %>
```

O si no desea procesar un campo `authenticity_token`:

```ruby
<%= form_for @invoice, url: external_url, authenticity_token: false do |f| %>
  Form contents
<% end %>
```



