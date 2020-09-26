---
layout: default
---

# Git - algunos extras
Cerramos la parte dedicada a repositorios de código comentando brevemente algunas características de Git de las que no hablamos hasta ahora.


## Tag
El concepto de **tag** es anterior a Git. Un tag es una "foto" del código en un momento determinado, que marca un evento, típicamente que el código tagueado es exactamente el que se desplegó en algún ambiente. Cada tag tiene un nombre/identificador, que puede ser el de release o el de build.

Para representar este concepto, Git define tags, que son _referencias a commits_. Sí, exactamente igual que los branches. 
La diferencia es conceptual: un tag es (en principio) fijo: una vez que se definió un tag que señala un determinado commit, ese tag no debería moverse salvo imprevistos.  
Haciendo una analogía con lenguajes de programación, los commits forman el espacio de memoria, un branch es análogo a una variable, un tag es análogo a una constante.

Para definir tags, Git define el comando `git tag`, que _actúa sobre el repo local_. Para pasar tags definidos localmente a un repo remoto, hay que usar la opción `--tags` de `git push`. 

En la misma documentación sobre `git tag`, o googleando, se encuentra fácilmente la forma de "mover" un tag en caso de necesidad.


## Cherry-pick
El comando `git cherry-pick`, permite aplicar los mismos cambios registrados en un commit _cualquiera_, sobre el `HEAD`. Como resultado, se agrega un nuevo commit.  
Este comando es tan potente como peligroso, porque deja duplicados de commits que generan ruido en el repo.


## Bisect
Imaginemos que queremos buscar el commit donde se introdujo un cambio. Git nos permite, con santa paciencia, ir checkouteando en una copia local los commits de un repo uno por uno, buscando en cada uno si está el cambio buscado.  
Obviamente, si el repo tiene _muchos_ commits, pongamos 1000, esto se vuelve impracticable. 

Mucho más práctico sería implementar una **búsqueda binaria**: se checkoutea el commit "del medio" (en este ejemplo el 500). Si el cambio está, entonces se introdujo antes; si no, después (o en el mismo commit "del medio"). Esto divide por dos la cantidad de commits posibles: se repite la operación de mirar el commit "del medio" y saber si se introdujo antes o después, hasta que obtenemos el primer commit con el cambio (porque miramos el inmediato anterior y no lo tiene).  
Como tal vez hayan visto en la facultad, esto reduce la cantidad de análisis de commits necesarios a 10.

Git incluye el comando `git bisect`, que ayuda a hacer exactamente este proceso. Los detalles, por favor, mirémoslos en la documentación.




