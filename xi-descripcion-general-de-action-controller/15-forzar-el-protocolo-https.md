# 15- Forzar el protocolo HTTPS

En ocasiones es posible que desee forzar a un controlador en particular a que sólo sea accesible a través de un protocolo HTTPS por razones de seguridad. Puede utilizar el método `force_ssl` en su controlador para hacer cumplir lo siguiente:

```ruby
class DinnerController
  force_ssl
end
```

Al igual que los filtros, también podría pasar `:only` y `:except` para hacer cumplir la conexión segura sólo a acciones específicas:

```ruby
class DinnerController
  force_ssl only: :cheeseburger
  # or
  force_ssl except: :cheeseburger
end
```

Ten en cuenta que si te encuentras añadiendo `force_ssl` a muchos controladores, quizás quieras obligar a toda la aplicación a usar `HTTPS` en su lugar. En ese caso, puede configurar config.`force_ssl` en su archivo de entorno.

