---
layout: default
---

# "Deshaciendo" o "modificando" cambios - reset, commit --amend, revert
Al agregar un commit, podemos considerar que el branch actual y `HEAD` _avanzan_ un paso: se mueven "hacia adelante", apuntando al commit recién creado.  
¿Cómo se hace para lograr un **retroceso** de un branch, para "arrepentirnos" del último commit registrado, borrarlo y volver el tip del branch y `HEAD` "un commit para atrás"?  
Para ser precisos, lo que queremos es que `HEAD` y el branch vuelvan al _parent_ del commit actual.

Mediante `git checkout`, se puede mover `HEAD` al parent ... o a cualquier commit que querramos.
Como toda la información commiteada está en el repo, no se pierde nada salvo lo que pudiera estar _solamente_ en el working tree o en el stage area: lo que ya se commiteó, no se va a perder por andar moviendo `HEAD`. Si vuelvo a mover `HEAD` a un commit conveniente, las versiones actualizadas van a volver a aparecer en el working tree. 
Esto nos sirve para consultar en qué estado estaba el repo en un determinado commit, pero no para mover branches.

Para mover al mismo tiempo el `HEAD` y el branch checkouteado, está `git reset`. Este comando tiene variantes interesantes para ajustar la comprensión sobre los distintos [espacios de Git](./git-espacios). También nos sirve para quitar elementos del stage area. Un super-comando el `git reset`.


## Deshaciendo el último commit - los tres modos de git reset
Un escenario típico de uso de `git reset` es cuando poco después de haber agregado un commit, nos damos cuenta de que nos apresuramos, o que no lo armamos en forma correcta.  
Por lo tanto, queremos que el branch actual y `HEAD` vuelvan "un commit para atrás", o sea

```
git reset HEAD^
```

El efecto que tiene este comando **en el repo** es el siguiente.

![efecto de un git reset en el repo](./images/reset-in-repo.jpg)

(qué pasa con el commit C8 lo vamos a ver más adelante).

Nos falta decidir cómo queremos que queden el working tree y la stage area. Dicho de otra forma, qué queremos hacer con los _cambios_ registrados en el commit que estamos "tirando para atrás". 
Hay tres opciones, a cada una le corresponde una variante del comando `git reset`. 
En resumen, tenemos:
- `git reset --soft HEAD^`: los cambios quedan en el stage area y en el working tree.
- `git reset --mixed HEAD^`: los cambios quedan solamente en el working tree, hay que agregar al stage area lo que se quiera commitear.
- `git reset --hard HEAD^`: los cambios no quedan en ningún lado. En principio, se pierden. En realidad, no se pierden del-todo-del-todo, esto tiene que ver con el status en que queda el commit C8 ... hablaremos sobre esto más adelante.

El default es `--mixed`, o sea, `git reset HEAD^` es equivalente a `git reset --mixed HEAD^`.

Contamos las tres variantes con gráficos que suponen que el commit descartado agrega el archivo `3.txt` a un repo que incluía `1.txt` y `2.txt`.

![reset --soft](./images/reset-soft-effect.jpg)

![reset --mixed](./images/reset-mixed-effect.jpg)

![reset --hard](./images/reset-hard-effect.jpg)

**Importante**  
Notar que la _única_ variante que modifica el working tree es `git reset --hard`.

### Algunos ejercicios
Sobre un repositorio, generar un commit que modifique dos archivos.
Después, deshacer ese commit (usando `git reset`) y reemplazarlo por dos commits secuenciales, uno para los cambios de cada archivo.

Si en un repositorio en este estado (notar en particular cuál es el branch actual)  
![Ejercicio sobre reset con varios branches](./images/reset-exercise-many-branches.jpg)
se hace un `git reset --hard HEAD^`, ¿se pierden los cambios registrados en C8? Si no se pierden, ¿cómo se puede hacer para verlos?

Sobre un repositorio, generar un commit que agregue un archivo y elimine otro. 
Pensar cómo quedaría el working tree, y cuál sería el resultado de `git status`, luego de un `git reset HEAD^`. Lo mismo con `git reset --hard HEAD^`. Después probar y verificar.  
**Nota**:  
para que sea fácil hacer las dos pruebas, después del último commit y antes de probar `git reset`, copiar todo el working tree a otra carpeta _incluyendo archivos ocultos_ y con _copia recursiva_. Si se hace así, la nueva carpeta también va a ser un repositorio Git. Ahora se puede hacer una prueba en cada carpeta.


## Reset en cualquier dirección
¿Qué pasa si se hace `git reset --soft HEAD~2`?  
El efecto _en el repositorio_ es el imaginable: el branch actual y `HEAD` se mueven **dos** commits hacia atrás.  
_En el working tree y en la stage area_, se van a _consolidar_ los cambios de los dos commits.


OK, entonces `git reset` permite ir hacia atrás todo lo que querramos. ¿Permitirá ir _hacia adelante_?
Sí, es posible hacer eso. En rigor, `git reset` permite cambiar el branch actual y `HEAD` a **cualquier commit**. Si se usa `git reset --soft`, calcula la diferencia entre el commit actual y el que se indica, y con eso arma la stage area.


### A probar
Generar un repositorio de esta forma  
![Ejercicio sobre reset con varios branches](./images/exercise-four-commits-three-branches.jpg)  
en el cual cada branch va agregando líneas a un único archivo de texto.
**OJO** - prestar atención a cuál tiene que ser el branch actual.  
En esta situación, hacer `git reset task02`. Pensar qué diferencia hay entre el working tree y `HEAD`. Verificarlo mediante `git diff`.  
Para volver a la situación anterior, alcanza con `git reset <id-del-commit-C2>`. Verificarlo.

Sobre la misma situación inicial, probar con `git reset --hard task02`. Pensar cómo queda el working tree y qué cambios muestra `git status`. Verificar.  
Pregunta: a partir de la situación posterior al `git reset --hard`, ¿se puede volver exactamente a la situación inicial? ¿Cómo?

A partir de la misma situación inicial, hacer las operaciones necesarias para "invertir" los tips de los branches `task01` y `task02`. O sea, que el repo quede así.
![Inversión de branches - objetivo](./images/exercise-four-commits-three-branches-goal.jpg)  


## Correcciones sencillas al último commit - commit --amend
Volvamos al escenario con el que arrancamos esta página: se armó mal el último commit. Puede ser p.ej. que nos hayamos olvidado de agregar un cambio, y/o que querramos cambiar el mensaje.

Para estos casos, una opción más sencilla es el `git commit --amend`. Si en un repositorio de esta forma  
![git commit --amend - antes](./images/just-four-commits.jpg)  
hacemos `git commit --amend -m "C4.1"`, obtenemos esto
![git commit --amend - después - fantasía](./images/change-commit-name.jpg)  

Hagámoslo, mirando el resultado de `git log --oneline` antes y después.
![git commit --amend - ingenuo](./images/commit-amend-naive.jpg)  

Bien, el nombre del commit cambió efectivamente de `C4` a `C4.1`.


## Git no pierde nada - reflog
Mirando con detenimiento el último screenshot de la consola, podemos descubrir que hay algo más que cambió, además del nombre del commit.

![git commit --amend - la otra diferencia](./images/commit-amend-with-remarks.jpg)  

La otra diferencia está en el _id del commit_. ¿Puede cambiar el id de un commit? No, los id no cambian. Lo que ocurre es que en realidad, el resultado de la operación `git commit --amend` es la creación de un _nuevo_ commit. La única diferencia con el `commit` normal es que el _parent_ del nuevo commit no es el commit actual, sino el parent del commit actual. En realidad, el repo después del `commit --amend` anterior queda así.  
![git commit --amend - después - real](./images/change-commit-name-real.jpg)  

El commit C4 queda "huérfano", en el sentido que no se puede acceder a partir de ningún branch. Pero sigue existiendo, y sigue teniendo el id que empieza con `5425b87`. Por ejemplo, si hacemos 
`git checkout 5425b87`
el `HEAD` va a apuntar (en modo "detached") al commit C4.

Esto nos sirve para "arrepentirnos" del cambio de nombre y volver a C4. Alcanza con volver a `master` (`git checkout master`) y después hacer
```
git reset --hard 5425b87
```
Ahora el commit "huérfano" es el C4.1.  
![git commit --amend - se revierte](./images/commit-amend-reversed.jpg)  

Si ya "perdimos" el id del commit que quedó huérfano, tenemos otra fuente de información que es el **reflog**. Este es un registro de cada operación que se hizo en un repositorio, que indica el id del `HEAD` en cada paso. Después del `git commit --amend`, este es el `reflog`.

![reflog después de un commit --amend](./images/reflog-after-amend.jpg)  

Vemos que aparecen los ids de todos los commits, incluso el huérfano.

Git incluye un garbage collector que limpia los repositorios de commits huérfanos. Pero para que limpie un commit, si entendí bien, hay que eliminar sus referencias en el reflog. A partir de estas ideas se puede explorar una parte de Git que está ampliamente fuera de mi alcance. Sólo hice una pequeña prueba basada en [esta respuesta en Stack Overflow](https://stackoverflow.com/a/29203553/7405996) que parece haber funcionado. 


### Más ejercicios
Plantear un escenario en el que hacer un `git reset` genera que _dos_ commits queden huérfanos.

A partir del escenario posterior al `git commit --amend`, o sea  
![git commit --amend - después - real](./images/change-commit-name-real.jpg)   
realizar las operaciones necesarias para que el repositorio quede así
![git ejercicio con commit huérfano - objetivo](./images/change-commit-name-exercise.jpg)   


A partir de un repositorio de esta forma  
![git ejercicio de commit --amend feo - escenario](./images/four-commits-two-branches.jpg)   
cambiarle el nombre al commit C2 usando `commit --amend`. Verificar usando `git log` que efectivamente cambió el nombre. Después hacer `git checkout master` y luego `git log`. Probablemente haya algo sorprendente en el resultado. Analizarlo y obtener conclusiones sobre el carácter delicado del `commit --amend`.



