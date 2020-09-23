---
layout: default
---

# Pull requests
Hasta ahora, describimos algunas características y comandos de Git con un enfoque _técnico_.
Partimos de la concepción de un repo como una red de commits, estudiamos con algún detalle el efecto de distintos comandos en esa red, y analizamos la relación entre un repo local y su/s remoto/s.  
Al final de la página anterior, empezamos a hablar de _escenarios de uso_, específicamente para pensar en distintos casos, las consecuencias de usar merge o rebase.

En el resto del material sobre Git, vamos a enfocarnos más en el _uso_ de Git en un proyecto. 
En esta página vamos a comentar los pull requests; en la siguiente se va a describir un posible _modelo de branching_ para un proyecto.


## Contexto de los pull requests - integración de código
No es nuevo para ustedes que cada tarea se trabaja (o debería trabajarse) en un branch separado.
Cuano se considera que la tarea está terminada, el código se va integrando (o "promoviendo") a otros branches que consolidan el trabajo de distintas personas o equipos, y/o se deployan en distintos ambientes "intermedios"(de tests automáticos, QA, UAT) hasta llegar al objetivo final, que es el branch que se depliega en ambientes productivos.

Desde un punto de vista técnico, y hablamos bastante sobre cómo implementar una integración de código de un branch de tarea en un branch de integración, utilizando Git. Digamos que queremos integrar los cambios registrados en el branch `task01`, sobre el task `dev`. Una forma posible consiste en la siguiente secuencia de comandos y acciones.
1. `git checkout task01`
1. `git pull`
1. `git checkout dev`
1. `git merge task01`, resolviendo eventualmente los conflictos que requieren resolución manual.
1. `git push <remoto> dev`

Esto lo hace un integrante del equipo del proyecto, que está encargado de las integraciones.
Después de hacer el `pull` de `task01`, puede revisar los cambios, para lo que puede utilizar `git diff` (para los cambios en el código) y `git log` (para ver los commits y sus descripciones), o bien usar herramientas gráficas como p.ej. GitKraken.  
El `git log` (o su equivalente en herramientas gráficas) le permiten saber quién hizo cada commit, para poder hacer consultas sobre el código que está revisando.


## Qué es un pull request - el concepto
Los PR son un concepto agregado al modelo de Git, que _formaliza el proceso de integración_.
Un PR permite visualizar cada evento de integración: le da una interfaz que permite consultar los cambios a integrar, se puede pedir opinión a integrantes del equipo, a partir de lo que se pueden generar intercambios. Se pueden consultar los PR en proceso, los que una determinada persona tiene pendientes de revisión, incluso los ya cerrados.

Un PR representa al proceso descripto antes, en el que una persona se baja un branch con cambios, lo revisa y, o bien lo aprueba y mergea (o rebasea) los cambios sobre el branch de integración, o bien lo rechaza, con lo cual el branch con los cambios "muere ahí".  
Las **únicas** diferencias de un PR son
1. que en lugar de ser un procso local en el equipo de la persona que integra, queda registrado, y varias personas pueden ver los cambios propuestos y opinar sobre ellos. Incluso, se puede indicar que para aceptar un PR, debe haber una cantidad mínima de revisores que aprueben los cambios.
1. que el merge, o el rebase, se hacen _directamente_ sobre un repositorio remoto; la operación nunca pasa por un repo local. Obviamente que una vez mergeado un PR, se puede hacer `pull` para ver los resultados en un repo local. Si se hace merge para la integración, se va a generar el merge commit en el repo remoto, y mediante el `pull` se copia en el repo local.

El nombre "pull request" se refiere a que las personas encargadas del branch con los cambios, les piden a los responsables de integrar, que se _pulleen_ el branch con cambios, lo miren, y si están de acuerdo, lo mergeen. El pull es el paso 2 del proceso local descripto más arriba.