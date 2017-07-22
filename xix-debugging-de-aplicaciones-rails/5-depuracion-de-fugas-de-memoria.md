# 5- Depuración de fugas de memoria

Una aplicación Ruby \(en Rails o no\), puede perder la memoria - ya sea en el código Ruby o en el nivel de código C.

En esta sección, aprenderá cómo encontrar y corregir dichas filtraciones utilizando una herramienta como `Valgrind`.

### 5.1 Valgrind

[Valgrind](http://valgrind.org/) es una aplicación para detectar fugas de memoria basadas en C y condiciones de carrera.

Hay herramientas de Valgrind que pueden detectar automáticamente muchos problemas de gestión de memoria y de subprocesos, y perfilar sus programas en detalle. Por ejemplo, si una extensión C del intérprete llama a `malloc()` pero no llama correctamente a `free()`, esta memoria no estará disponible hasta que finalice la aplicación.

Para obtener más información sobre cómo instalar Valgrind y utilizarla con Ruby, consulte [Valgrind y Ruby](https://blog.evanweaver.com/2008/02/05/valgrind-and-ruby/) por Evan Weaver.



