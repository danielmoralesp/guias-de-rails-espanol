# 4. Ejecutando Callbacks

Los siguientes métodos disparan callbacks:

```ruby
create
create!
decrement!
destroy
destroy!
destroy_all
increment!
save
save!
save(validate: false)
toggle!
update_attribute
update
update!
valid?
```

Adicionalmente, el callback `after_find` es disparado por los siguientes métodos de búsqueda:

```ruby
all
first
find
find_by
find_by_*
find_by_*!
find_by_sql
last
```

El callback `after_initialize` es lanzado cada vez que un nuevo objeto de la clase es instanciado. 

Los métodos `find_by_*` y `find_by_*!` son finders dinámicos generados automáticamente para todos los atributos. Aprende más acerca de ellos en la Sección de finders dinámicos



