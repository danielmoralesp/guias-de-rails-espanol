# 2.4 Uso de los headers para crear respuestas Header-Only

El método `head` se puede utilizar para enviar respuestas con sólo encabezados al navegador. El método `head` acepta un número o símbolo \(véase tabla de referencia\) que representa un código de estado `HTTP`. El argumento `options` se interpreta como un `hash` de nombres de encabezado y valores. Por ejemplo, sólo puede devolver un encabezado de error:

```ruby
head :bad_request
```

Esto produciría el siguiente encabezado:

```ruby
HTTP/1.1 400 Bad Request
Connection: close
Date: Sun, 24 Jan 2010 12:15:53 GMT
Transfer-Encoding: chunked
Content-Type: text/html; charset=utf-8
X-Runtime: 0.013483
Set-Cookie: _blog_session=...snip...; path=/; HttpOnly
Cache-Control: no-cache
```

O puede utilizar otros encabezados `HTTP` para transmitir otra información:

```ruby
head :created, location: photo_path(@photo)
```

Lo que produciría:

```ruby
HTTP/1.1 201 Created
Connection: close
Date: Sun, 24 Jan 2010 12:16:44 GMT
Transfer-Encoding: chunked
Location: /photos/1
Content-Type: text/html; charset=utf-8
X-Runtime: 0.083496
Set-Cookie: _blog_session=...snip...; path=/; HttpOnly
Cache-Control: no-cache
```



