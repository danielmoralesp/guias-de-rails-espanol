# 9- Actualización de versiones antiguas de Rails

Existen algunos problemas al actualizar desde Rails 3.0 o Rails 2.x. El primero es mover los archivos de `public/` a las nuevas ubicaciones. Consulte la organización de activos \(vista arriba\) para obtener orientación sobre las ubicaciones correctas para diferentes tipos de archivos.

A continuación evitará duplicar archivos JavaScript. Puesto que jQuery es la biblioteca predeterminada de JavaScript de Rails 3.1 en adelante, no es necesario copiar` jquery.js` en `app/assets` y se incluirá automáticamente.

La tercera es la actualización de los distintos archivos de entorno con las opciones predeterminadas correctas.

En `application.rb`:

```ruby
# Version of your assets, change this if you want to expire all your assets
config.assets.version = '1.0'
 
# Change the path that assets are served from config.assets.prefix = "/assets"
```

En `development.rb`:

```ruby
# Expands the lines which load the assets
config.assets.debug = true
```

Y en `production.rb`:

```ruby
# Choose the compressors to use (if any)
config.assets.js_compressor = :uglifier
# config.assets.css_compressor = :yui
 
# Don't fallback to assets pipeline if a precompiled asset is missed
config.assets.compile = false
 
# Generate digests for assets URLs.
config.assets.digest = true
 
# Precompile additional assets (application.js, application.css, and all
# non-JS/CSS are already added)
# config.assets.precompile += %w( admin.js admin.css )
```

En Rails 4 y superiores ya no se establecen valores de configuración predeterminados para Sprockets en `test.rb`, por lo que `test.rb` ahora requiere la configuración de Sprockets. Los valores predeterminados anteriores en el entorno de prueba son: `config.assets.compile = true, config.assets.compress = false, config.assets.debug = false and config.assets.digest = false`.

Lo siguiente también se debe agregar a su Gemfile:

```ruby
gem 'sass-rails',   "~> 3.2.3"
gem 'coffee-rails', "~> 3.2.1"
gem 'uglifier'
```



