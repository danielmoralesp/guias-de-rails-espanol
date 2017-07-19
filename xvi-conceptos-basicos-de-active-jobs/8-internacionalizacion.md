# 8- Internacionalización

Cada job utiliza el conjunto `I18n.locale` cuando se crea el trabajo. Es útil si envía mensajes de correo electrónico de forma asíncrona:

```ruby
I18n.locale = :eo
 
UserMailer.welcome(@user).deliver_later # Email will be localized to Esperanto.
```



