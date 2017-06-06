# 6.2 AtomFeedHelper



### 6.2.1 atom\_feed

Este helper facilita la creación de un feed `Atom`. He aquí un ejemplo de uso completo:

`config/routes.rb`

```ruby
resources :articles
```

`app/controllers/articles_controller.rb`

```ruby
def index
  @articles = Article.all
 
  respond_to do |format|
    format.html
    format.atom
  end
end
```

`app/views/articles/index.atom.builder`

```ruby
atom_feed do |feed|
  feed.title("Articles Index")
  feed.updated(@articles.first.created_at)
 
  @articles.each do |article|
    feed.entry(article) do |entry|
      entry.title(article.title)
      entry.content(article.body, type: 'html')
 
      entry.author do |author|
        author.name(article.author_name)
      end
    end
  end
end
```



