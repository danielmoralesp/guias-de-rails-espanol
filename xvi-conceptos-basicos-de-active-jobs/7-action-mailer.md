# 7- Action Mailer

Uno de los trabajos más comunes en una aplicación web moderna es el envío de correos electrónicos fuera del ciclo de solicitud y respuesta, por lo que el usuario no tiene que esperar. Active Job se integra con Action Mailer para que pueda enviar correos electrónicos de manera asincrónica:

```ruby
# Si quieres enviar el correo electrónico ahora usa #deliver_now
UserMailer.welcome(@user).deliver_now
 
# Si desea enviar el correo electrónico a través del active job use #deliver_later
UserMailer.welcome(@user).deliver_later
```



