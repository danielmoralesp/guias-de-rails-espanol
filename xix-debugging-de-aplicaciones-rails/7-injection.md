# 7- Injection

> Injection es una clase de ataques que introducen código malicioso o parámetros en una aplicación web para ejecutarlo dentro de su contexto de seguridad. Los ejemplos prominentes de la inyección son secuencias de comandos entre sitios \(XSS\) e inyección de SQL.

La inyección es muy complicada, porque el mismo código o parámetro puede ser malicioso en un contexto, pero totalmente inofensivo en otro. Un contexto puede ser un lenguaje de programación, consulta o de programación, el shell o un método Ruby / Rails. Las secciones siguientes cubrirán todos los contextos importantes donde pueden ocurrir ataques de inyección. La primera sección, sin embargo, cubre una decisión arquitectónica en relación con la inyección.



