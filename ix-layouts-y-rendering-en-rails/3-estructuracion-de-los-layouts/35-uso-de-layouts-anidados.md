# 3.5 Uso de layouts anidados

Es posible que su aplicación requiera un layout que difiera ligeramente de su layout de aplicación habitual para dar soporte a un controlador en particular. En lugar de repetir el layout principal y editarlo, puede lograr esto utilizando layouts anidados \(a veces llamados sub-templates\). He aquí un ejemplo:

Supongamos que tiene el siguiente layout de `ApplicationController`:

`app/views/layouts/application.html.erb`

```ruby
<html>
<head>
  <title><%= @page_title or "Page Title" %></title>
  <%= stylesheet_link_tag "layout" %>
  <style><%= yield :stylesheets %></style>
</head>
<body>
  <div id="top_menu">Top menu items here</div>
  <div id="menu">Menu items here</div>
  <div id="content"><%= content_for?(:content) ? yield(:content) : yield %></div>
</body>
</html>
```

En las páginas generadas por `NewsController`, desea ocultar el menú superior y agregar un menú de la derecha:

`app/views/layouts/news.html.erb`

```ruby
<% content_for :stylesheets do %>
  #top_menu {display: none}
  #right_menu {float: right; background-color: yellow; color: black}
<% end %>
<% content_for :content do %>
  <div id="right_menu">Right menu items here</div>
  <%= content_for?(:news_content) ? yield(:news_content) : yield %>
<% end %>
<%= render template: "layouts/application" %>
```

Eso es. La vista News utilizarán el nuevo layout, ocultando el menú superior y añadiendo un nuevo menú derecho dentro del div "`content`".

Hay varias maneras de obtener resultados similares con diferentes esquemas de sub-plantillas usando esta técnica. Tenga en cuenta que no hay límite en los niveles de anidamiento. Puede usar el método `ActionView::render` a través de la plantilla de render: '`layouts/news`' para basar un nuevo layout en el layout de news. Si está seguro de que no quiere "sub-templear" el layout de noticias, puede reemplazar el `content_for?(:news_content) ? yield(:news_content) : yield` por un simple `yield`. 

