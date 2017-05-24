# 7. Callbacks Relacionados

Los Callbacks trabajan a través del modelo de relaciones, y pueden también ser definidos por ellos. Supón un ejemplo donde un usuario tiene muchos artículos. Un artículo de usuario debería ser borrado si el usuario es borrado. Vamos a añadir un callback `after_destroy` al modelo `User` de esta relación con el modelo `Article`:

```ruby
class User < ActiveRecord::Base
  has_many :articles, dependent: :destroy
end
 
class Article < ActiveRecord::Base
  after_destroy :log_destroy_action
 
  def log_destroy_action
    puts 'Artículo borrado'
  end
end
 
>> user = User.first
=> #<User id: 1>
>> user.articles.create!
=> #<Article id: 1, user_id: 1>
>> user.destroy
Artículo borrado
=> #<User id: 1>
```



