# 6- Almacén de caché de recursos

De forma predeterminada, Sprockets almacena en caché recursos en `tmp/cache/assets` en entornos de desarrollo y producción. Esto se puede cambiar de la siguiente manera:

```ruby
config.assets.configure do |env|
  env.cache = ActiveSupport::Cache.lookup_store(:memory_store,
                                                { size: 32.megabytes })
end
```

Para deshabilitar el almacén de caché de recursos:

```ruby
config.assets.configure do |env|
  env.cache = ActiveSupport::Cache.lookup_store(:null_store)
end
```



