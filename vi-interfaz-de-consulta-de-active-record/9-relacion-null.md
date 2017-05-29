# 9.  Relación NULL

El método `none` devuelve una relación encadenable sin registros. Cualquier condición subsiguiente encadenada a la relación devuelta continuará generando relaciones vacías. Esto es útil en escenarios donde se necesita una respuesta encadenable a un método o un scope que podría devolver resultados cero.

```ruby
Article.none # Devuelve una relación vacía y no activa ninguna consulta.
```

```ruby
# Se espera que el método visible_articles a continuación devuelva una Relación.
@articles = current_user.visible_articles.where(name: params[:name])
 
def visible_articles
  case role
  when 'Country Manager'
    Article.where(country: country)
  when 'Reviewer'
    Article.published
  when 'Bad User'
    Article.none # => devolviendo [] o nil interrumpe el código de llamada en este caso
  end
end
```



