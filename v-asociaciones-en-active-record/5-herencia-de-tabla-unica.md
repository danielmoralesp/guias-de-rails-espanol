# 5. Herencia de tabla única

A veces, es posible que desee compartir campos y comportamiento entre diferentes modelos. Digamos que tenemos modelos de coches, motos y bicicletas. Queremos compartir los campos de color y precio y algunos métodos para todos ellos, pero teniendo un comportamiento específico para cada uno, y controladores separados también. 

Rails hace esto muy fácil. Primero, vamos a generar la base Modelo del vehículo:

```ruby
$ rails generate model vehicle type:string color:string price:decimal{10.2}
```

¿Ha notado que estamos agregando un campo de `type`? Dado que todos los modelos se guardarán en una sola tabla de base de datos, Rails guardará en esta columna el nombre del modelo que se está guardando. En nuestro ejemplo, esto puede ser `Car`, `Motorcycle` o `Bicycle`. STI no funcionará sin un campo `type` en la tabla. 

A continuación, vamos a generar los tres modelos que heredan de Vehículo. Para ello, podemos usar la opción `--parent=PARENT`, que generará un modelo que hereda del padre especificado y sin migración equivalente \(ya que la tabla ya existe\). 

Por ejemplo, para generar el modelo de `Car`:

```ruby
$ rails generate model car --parent=Vehicle
```

El modelo generado se verá así:

```ruby
class Car < Vehicle
end
```

Esto significa que todos los comportamientos agregados a Vehículo también están disponibles para `Car`, como asociaciones, métodos públicos, etc. 

La creación de un `Car` lo guardará en la tabla de `Vehicle` con `Car` como el campo `Type`:

```ruby
Car.create(color: 'Red', price: 10000)
```

Generará el siguiente `SQL`:

```ruby
INSERT INTO "vehicles" ("type", "color", "price") VALUES ('Car', 'Red', 10000)
```

Consultar los registros de automóviles solo buscará vehículos que sean automóviles:

```ruby
Car.all
```

Ejecutará una consulta como:

```ruby
SELECT "vehicles".* FROM "vehicles" WHERE "vehicles"."type" IN ('Car')
```



