# 2. Un vistazo a los Callbacks

Los Callbacks son métodos que son llamados en ciertos momentos del ciclo de vida de un objeto. Con los callbacks es posible escribir código que se ejecutará siempre que un objeto Active Record es creado, guardado, actualizado, borrado, validado o cargado desde la base de datos. 



### 2.1 Registro de Callbacks 

Con el objeto de utilizar algunos callbacks disponibles, necesitas registrarlos. Puedes implementar los callbacks como cualquier método normal y utilizar un método de clase macro-style para registrarlos como callbacks:

```ruby
class User < ActiveRecord::Base
  validates :login, :email, presence: true
 
  before_validation :ensure_login_has_a_value
 
  protected
    def ensure_login_has_a_value
      if login.nil?
        self.login = email unless email.blank?
      end
    end
end
```

Los métodos de clase macro-style pueden también recibir un bloque. Considera utilizar este estilo si el código dentro de tu bloque está utilizando es tan corto que llevará una sola línea:

```ruby
class User < ActiveRecord::Base
  validates :login, :email, presence: true
 
  before_create do
    self.name = login.capitalize if name.blank?
  end
end

class User < ActiveRecord::Base
  before_validation :normalize_name, on: :create
 
  # :on takes an array as well
  after_validation :set_location, on: [ :create, :update ]
 
  protected
    def normalize_name
      self.name = self.name.downcase.titleize
    end
 
    def set_location
      self.location = LocationService.query(self)
    end
end
```

Está considerada una buena práctica declarar los métodos callback como privados o protegidos. Si los dejas públicos, pueden ser llamados desde fuera del modelo y violar el principio de encapsular del objeto.

