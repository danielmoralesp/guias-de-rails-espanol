# 3. Callbacks Disponibles

Aquí hay una lista con todos los callbacks Active Record, listadas en el mismo orden en el cual serán llamados durante la ejecución de las respectivas operaciones:



### 3.1 Creando un Objeto

```ruby
before_validation
after_validation
before_save
around_save
before_create
around_create
after_create
after_save
after_commit/after_rollback
```

### 3.2 Actualizando un Objecto

```ruby
before_validation
after_validation
before_save
around_save
before_update
around_update
after_update
after_save
after_commit/after_rollback
```

### 3.3 Destruyendo un Objecto

```ruby
before_destroy
around_destroy
after_destroy
after_commit/after_rollback
```

`after_save` se ejecuta tanto en la creación como en la actualización, pero siempre `after` los callbacks más específicos `after_create` y `after_update`, no cambia el orden en el cual las llamadas macro fueron ejecutadas.



### 3.4 after\_initialize y after\_find 

El callbak `after_initialize` será llamado siempre que un objeto Active Record es instanciado, tanto por el uso directo de `new` o cuando un objeto es cargado desde la base de datos. Esto puede ser de utilidad cuando se tiene la necesidad de directamente sobrescribir un método Active Record `initialize`. 

El callback `after_find` será llamado siempre que Active Record cargue un registro desde la base de datos. `after_find` es llamado antes de `after_initialize` si ambos están definidos. 

Los callbacks `after_initialize` y `after_find` no tienen contrapartidas `before_*` , pero ellos pueden ser registrados como cualquier otro callback Active Record.

```ruby
class User < ActiveRecord::Base
  after_initialize do |user|
    puts "You have initialized an object!"
  end
 
  after_find do |user|
    puts "You have found an object!"
  end
end
 
>> User.new
You have initialized an object!
=> #<User id: nil>
 
>> User.first
You have found an object!
You have initialized an object!
=> #<User id: 1>
```



### 3.5 after\_touch 

El callback `after_touch` será llamado si un objeto Active Record es tocado.

```ruby
class User < ActiveRecord::Base
  after_touch do |user|
    puts "Has tocado un objeto"
  end
end
 
>> u = User.create(name: 'Kuldeep')
=> #<User id: 1, name: "Kuldeep", created_at: "2013-11-25 12:17:49", updated_at: "2013-11-25 12:17:49">
 
>> u.touch
Has tocado un objeto
=> true
```

Este puede ser utilizado junto con `belongs_to`:

```ruby
class Employee < ActiveRecord::Base
  belongs_to :company, touch: true
  after_touch do
    puts 'Un empleado fue tocado'
  end
end
 
class Company < ActiveRecord::Base
  has_many :employees
  after_touch :log_when_employees_or_company_touched
private
  def log_when_employees_or_company_touched
    puts 'Employee/Company fueron tocados'
  end
end
 
>> @employee = Employee.last
=> #<Employee id: 1, company_id: 1, created_at: "2013-11-25 17:04:22", updated_at: "2013-11-25 17:05:05">
 
# triggers @employee.company.touch
>> @employee.touch
Employee/Company fueron tocados
Un empleado fue tocado
=> true
```







