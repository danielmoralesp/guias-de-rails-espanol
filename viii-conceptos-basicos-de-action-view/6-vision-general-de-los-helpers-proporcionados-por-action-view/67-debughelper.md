# 6.7 DebugHelper

Devuelve una etiqueta `pre` que tiene un objeto dumping de `YAML`. Esto crea una forma muy legible para inspeccionar un objeto.

```ruby
my_hash = { 'first' => 1, 'second' => 'two', 'third' => [1,2,3] }
debug(my_hash)
```

```ruby
<pre class='debug_dump'>---
first: 1
second: two
third:
- 1
- 2
- 3
</pre>
```



