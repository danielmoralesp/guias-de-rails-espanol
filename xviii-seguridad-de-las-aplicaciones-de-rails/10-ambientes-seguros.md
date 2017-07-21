# 10- Entornos seguros

Está fuera del alcance de esta guía informarle sobre cómo proteger el código y los entornos de su aplicación. Sin embargo, asegure su configuración de base de datos, por ejemplo. `config/database.yml` y su secret del lado del servidor, por ejemplo almacenado en `config/secrets.yml`. Es posible que desee restringir aún más el acceso, utilizando versiones específicas del entorno de estos archivos y cualquier otra que contenga información confidencial.

### 10.1 Secrets personalizados

Rails genera un `config/secrets.yml`. De forma predeterminada, este archivo contiene la base de datos secreta de la aplicación, pero también podría utilizarse para almacenar otros secretos, como claves de acceso para API externas.

Los secrets añadidos a este archivo son accesibles vía `Rails.application.secrets`. Por ejemplo, con el siguiente `config/secrets.yml`:

```ruby
development:
  secret_key_base: 3b7cd727ee24e8444053437c36cc66c3
  some_api_key: SOMEKEY
```

`Rails.application.secrets.some_api_key` devuelve SOMEKEY en el entorno de desarrollo.

Si desea que se genere una excepción cuando alguna tecla está en blanco, utilice la versión Bang:

```ruby
Rails.application.secrets.some_api_key! # => raises KeyError: key not found: :some_api_key
```



