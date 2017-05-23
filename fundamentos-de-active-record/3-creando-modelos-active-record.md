# 3. Creando Modelos Active Record

Es muy fácil crear modelos Active Record. Todo lo que necesitas hacer es una subclase de la clase `ActiveRecord::Base` y listo: 

```ruby
class Product < ActiveRecord::Base 
end
```

Esto creará una clase modelo Product, mapeada a la tabla products de la base de datos. Para hacer esto también tendrás que tener la posibilidad de mapear columnas de cada fila con los atributos de cada instancia del modelo. 

Supón que la tabla products fue creada utilizando una sentencia SQL como: 

```sql
CREATE TABLE products ( 
    id int(11) NOT NULL auto_increment, 
    name varchar(255), 
    PRIMARY KEY (id) 
);
```

Siguiendo el esquema de arriba, tendrías la capacidad de escribir código como el que sigue: 

```ruby
p = Product.new 
p.name = "Some Book" 
puts p.name # "Some Book"
```



