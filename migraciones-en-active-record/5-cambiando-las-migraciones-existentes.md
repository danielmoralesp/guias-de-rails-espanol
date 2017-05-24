# 5. Cambiando las Migraciones Existentes

Ocasionalmente podemos cometer un error cuando escribimos una migración. Si has ejecutado ya la migración entonces no puedes editarla y ejecutarla otra vez: Rails pensará que ya se ha ejecutado la migración y no hará nada cuando ejecutes otra vez `rake db:migrate`. Por eso debes revertir la migración \(por ejemplo con `rake db:rollback`\), edita la migración y luego escribe `rake db:migrate` para ejecutar la versión corregida. 

En general, editar una migración existente no es una buena idea. Crearás trabajo extra para ti mismo y tus colaboradores y causará mayores dolores de cabeza si la versión existente de una migración ha sido ya ejecutada en los servidores de producción. En su lugar, deberías escribir una nueva migración que realice los cambios que requieres. Editar una migración anterior con otra recién generada que no ha sido "commiteada" en la base de datos, controlamos mejor el código fuente \(que además en general no se ha propagado más allá de tu máquina de desarrollo\) y es relativamente inofensivo, y se convierte en una mejor practica. 

El método `revert` puede ser de mucha ayuda cuando escribes una nueva migración para deshacer migraciones anteriores en conjunto o en parte \(ver Revertir Migraciones Anteriores ver arriba\).



