# 6.11 JavaScriptHelper

Proporciona funcionalidad para trabajar con JavaScript en sus vistas.



### 6.11.1 escape\_javascript

Retornos de portador de escape y comillas simples y dobles para segmentos de JavaScript.



### 6.11.2 javascript\_tag

Devuelve una etiqueta JavaScript que envuelve el código proporcionado.

```ruby
javascript_tag "alert('All is good')"
```

```ruby
<script>
//<![CDATA[
alert('All is good')
//]]>
</script>
```



