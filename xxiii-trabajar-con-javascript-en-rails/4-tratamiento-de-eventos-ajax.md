# 4- Tratamiento de eventos Ajax

Estos son los diferentes eventos que se activan cuando se trata de elementos que tienen un atributo `data-remote`:

Todos los manejadores vinculados a estos eventos siempre pasan el objeto de evento como el primer argumento. La siguiente tabla describe los parámetros adicionales que se pasan después del argumento del evento. Por ejemplo, si los parámetros adicionales se enumeran como `xhr`, `settings`, entonces para acceder a ellos, se define su manejador con función \(`event`, `xhr`, `settings`\)

| Event name | Extra parameters | Fired |
| :--- | :--- | :--- |
| `ajax:before` |  | Antes de todo el negocio ajax, aborta si se detiene. |
| `ajax:beforeSend` | xhr, options | Antes de enviar la solicitud, se interrumpe si se detiene. |
| `ajax:send` | xhr | Cuando se envía la solicitud. |
| `ajax:success` | xhr, status, err | Después de la finalización, si la respuesta fue un éxito. |
| `ajax:error` | xhr, status, err | Después de la finalización, si la respuesta fue un error. |
| `ajax:complete` | xhr, status | Después de que la solicitud se ha completado, no importa el resultado. |
| `ajax:aborted:file` | elements | Si hay entradas de archivo non-blank, se interrumpe si se detiene. |

### 4.1 Eventos paralizables

Si detiene `ajax:before` o `ajax:beforeSend `devolviendo `false` del método `handler`, la solicitud Ajax nunca tendrá lugar. El evento `ajax:before` también es útil para manipular datos de un formulario antes de la serialización. El evento `ajax:beforeSend` también es útil para agregar encabezados de solicitud personalizada.

Si detiene el evento `ajax:aborted:file`, el comportamiento predeterminado de permitir que el navegador envíe el formulario por medios normales \(es decir, la presentación no-AJAX\) se cancelará y el formulario no se enviará en absoluto. Esto es útil para implementar su propia solución de carga de archivos AJAX.



