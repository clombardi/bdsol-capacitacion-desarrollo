---
layout: default
---

# Git
Como la enorme mayoría de los proyectos de desarrollo actuales, en Banco del Sol se usa [Git](https://git-scm.com/) como **repositorio de código**. 
Por lo tanto, ya están usando Git, y seguramente ya lo conocían antes de entrar en Banco del Sol. Cada vez cuesta más recordar cómo era el mundo del software antes de Git.  
Ya están familiarizados con varias operaciones que se pueden hacer sobre Git, entre ellas
- crear un repositorio en GitHub / GitLab / BitBucket.
- clonar un repositorio remoto en su equipo.
- generar un nuevo commit agregando código que escribieron.
- subir commits al repositorio remoto.
- bajarse las novedades de un repositorio remoto a su computadora.
- usar `git merge` para resolver conflictos.
- crear y comentar Pull Requests.


## Entonces ¿qué vamos a hacer?
Lo que saben ya lo saben, y no lo repetiremos aquí.  
Por otro lado, entre lo que seguramente saben, además del uso concreto, es que Git es un mundo, que incluye una cantidad impresionante de comandos (listé los que conozco, y llegué fácilmente a los 20), con _toneladas_ de opciones y variantes. 
Git se caracteriza por dar soporte para prácticamente cualquier situación que se pueda presentar en un desarrollo en el que participen muchas personas.

En estas sesiones apuntaremos a describir conceptos de Git desde un punto de vista más cercano a cómo está concebido, para ampliar nuestra comprensión de lo que hacemos en nuestro día a día, y agregar elementos que nos pueden ayudar a salir del paso en algunas ocasiones.

También vamos a experimentar con algunos comandos de `git` menos comunes, que nos pueden ayudar con algunas tareas.

Otro tema del que vamos a hablar son los _modelos de branching_, o sea, las decisiones sobre qué branches va a incluir cada repositorio (develop, release, task, etc.) y cómo va "viajando" el código de un branch a otro mediante PRs.


## Notas de color
¿Sabían que Git lo concibió Linus Torvalds en persona, no? 
También desarrolló la primer implementación, no sé si solo o con ayuda.

Git nació para soportar el desarrollo de Linux, proyecto colaborativo si los hay. 
De ahí que esté concebido para proyectos en los que colaboran muchas personas, que no necesariamente se conocen entre ellas. Hay un cuidado especial en brindar herramientas para comparar, analizar, sincronizar, las contribuciones de una cantidad importante de personas sobre un mismo proyecto.

Tal vez la característica que más se destacó de Git respecto de los repositorios más populares en el momento en que Git se hizo conocido (básicamente SVN) es lo fácil y liviano que es manejar **branches**.  
Con SVN, se pensaba mucho antes de desarrollar un feature en un branch separado. Git hace que esto sea mucho más sencillo.


## Git, GitHub, GitLab, Bitbucket
¿Qué es Git exactamente?

En primer lugar, es un programa, que se instala local. Permite crear y administrar repositorios locales.  
Git también define el formato en el que se almacena la información en un repositorio (o sea, el contenido de la subcarpeta `.git`  en cada carpeta en la que se crea o clona un repositorio).  
Finalmente, incluye el protocolo para comunicarse con repositorios remptos que se alojan en servidores.

Aunque una organización puede instalar y configurar sus propios servidores Git, actualmente es más común contratar Git como un servicio.
GitHub, GitLab y Bitbucket son los principales proveedores de estos servicios.
El servicio que ofrece cualquiera de estos proveedores no se limita a repositorios Git, incluyendo 
- la integración con issue trackers,
- la posibilidad de disparar acciones a partir de eventos en un repositorio (p.ej. mergear un PR o agregar commits en un determinado branch): realizar análisis de código, ejecutar tests, construir entregables, realizar el deploy típicamente en ambientes cloud.
- un sitio de documentación ("Wiki") asociado a cada repositorio.

En particular, GitHub también provee la posibilidad de publicar un repositorio, cuyos archivos están en formato Markdown, como un sitio Web estático. Es lo que se conoce como GitHub Pages. Sin ir más lejos, _este sitio_ está soportado por GitHub Pages.


## Referencias (que me parecen) útiles
Cierro la introducción con dos referencias que creo útiles sobre Git.

La primera es la serie de [tutoriales de Git de Atlassian](https://www.atlassian.com/git/tutorials). [Atlassian](https://www.atlassian.com/) es la empresa que mantiene Bitbucket y Jira, entre otros productos.  
De los tutoriales genéricos que vi, es el que considero más claro. Se entiende lo que explica, y va bastante en profundidad.

Otro sitio que me ha ayudado a entender Git es [Think like (a) Git](http://think-like-a-git.net/). Tiene unos cuantos años y cuenta sólo algunas de las cosas que se pueden hacer usando Git. Lo que me gusta es el enfoque, cuenta Git desde una perspectiva cercana a la concepción de la herramienta, haciendo el esfuerzo por ser digerible.