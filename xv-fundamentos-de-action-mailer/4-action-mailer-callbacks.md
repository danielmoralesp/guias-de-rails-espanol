# 4- Action Mailer Callbacks

Action Mailer permite especificar una `before_action`, `after_action` y `around_action`.

* Los filtros se pueden especificar con un bloque o un símbolo a un método en la clase de correo similar a los controladores.
* Puede utilizar un `before_action` para rellenar el objeto de correo con `defaults`, `delivery_method_options` o insertar encabezados y archivos adjuntos predeterminados.
* Usted podría usar un `after_action` para hacer una configuración similar a `before_action` pero usando variables de instancia establecidas en su acción de mailer.

```ruby

class UserMailer < ApplicationMailer
  after_action :set_delivery_options,
               :prevent_delivery_to_guests,
               :set_business_headers
 
  def feedback_message(business, user)
    @business = business
    @user = user
    mail
  end
 
  def campaign_message(business, user)
    @business = business
    @user = user
  end
 
  private
 
    def set_delivery_options
      # You have access to the mail instance,
      # @business and @user instance variables here
      if @business && @business.has_smtp_settings?
        mail.delivery_method.settings.merge!(@business.smtp_settings)
      end
    end
 
    def prevent_delivery_to_guests
      if @user && @user.guest?
        mail.perform_deliveries = false
      end
    end
 
    def set_business_headers
      if @business
        headers["X-SMTPAPI-CATEGORY"] = @business.code
      end
    end
end
```

* Filtros de correo abortan procesamiento adicional si el cuerpo se establece en un valor non-nil.





