---
layout: default
---

# La movida de microservicios

En los últimos años, aproximadamente desde 2013, la palabra **microservicio** viene escuchándose, cada vez más, en el ámbito de desarrollo de software.
Empezamos a leer/ver/presenciar artículos, posts y charlas, en donde se contaba que los microservicios estaban llegando para solucionar una serie de problemas provocados por "el monolito".  
En particular, nos enteramos que muchas organizaciones enormes fuertemente basadas en software, adoptaban enfoques basados en microservicios: Spotify, Netflix, Ebay, Uber, etc..

Esta difusión inicial impulsó la adopcion de la idea de microservicio en el desarrollo de software en muchas organizaciones ... entre ellas el Banco del Sol.

En esta sección, vamos a intentar describir de qué se trata esta "movida de microservicios", y cómo impacta en la organización y el día-a-día del desarrollo.  
Para una exposición más extensa y detallada, se puede consultar el libro que escribió Sam Newman [Building Microservices](https://www.oreilly.com/library/view/building-microservices/9781491950340/), o la [guía sobre microservicios en martinfowler.com](https://martinfowler.com/microservices/).


## Qué es eso de "microservicios"

Martin Fowler está entre los primeros difusores de los microservicios, definiéndolos como un "estilo arquitectural", p.ej. en [este artículo en martinfowler.com](https://martinfowler.com/articles/microservices.html).  

Creo que una buena aproximación para bajar a tierra la idea de microservicios, es pensarla como una serie de _pautas_, que guían tanto la forma de **desplegar** productos de software, como la forma de **desarrollarlos**.  
Aquí debe tenerse en cuenta que las pautas son _indicaciones_ para el trabajo, y no reglas estrictas. Tal como lo indica el libro "Building Microservices" recién citado

> Rules are for the obedience of fools and the guidance of wise men.

En este material, nos referiremos a los proyectos de desarrollo que siguen estas pautas, o al menos las principales, como _desarrollos basados en microservicios_.  
Obviamente, esto no genera una clasificación binaria, en la cual algunos proyectos están basados en microservicios y otros no, y estas son las únicas posibilidades.
En distintos proyectos se seguirán estas pautas en distinto grado, contemplando distintos recortes, y/o aplicándolas con distinto énfasis. 

Tal vez las pautas que se aprecian en forma más evidentes son los ligadas al _despliegue_: en lugar de tener un único producto que reúne toda la funcionalidad que necesita una organización, que debe ser primero generado y luego desplegado en bloque (este es el monolito), se recomienda contar definir muchos productos (los microservicios) que se despliegan por separado, y que se comunican mediante interfaces livianas, generalmente usando un lenguaje de intercambio de información como JSON, YAML o XML.  
Incluso, cada microservicio puede tener su base de datos propia, con lo cual no sólo se descentraliza el despliegue, sino también la gestión de datos.

Pero igualmente importante es el impacto que este cambio arquitectural tiene sobre el _desarrollo_. Vemos brevemente algunas consecuencias, que trataremos luego más extensamente.  
Al organizar el desarrollo en unidades separadas, se favorece el surgimiento de _equipos autónomos_, con mayor capacidad de decisión sobre el producto que desarrollan. 
A su vez, esto mejora la satisfacción, y a la vez el compromiso, de cada equipo respecto de sus productos.  
Además, se tiende a que los equipos sean _full-stack_, cambiando la organización tradicional ligada a tecnologías (equipos de BD, de lógica de negocio, de comunicaciones, de UI, etc.).


## Por qué surgieron - motivaciones

La propuesta de desarrollo basado en microservicios surgió como reacción a varios problemas concretos que aparecieron en muchos proyectos de desarrollo, en particular en aquellos de mediana o gran escala (o sea, que insumen decenas o cientos de años-hombre). Citando al libro "Building Microservices" ya mencionado.
> Microservices (...) weren't invented or described before the fact; they emerged as a trend, or a pattern, from real-world use.


### Problemas técnicos

Un problema particularmente relevante es la _dificultad para generar los desplegables_ en un proyecto organizado como un monolito, o sea, un único desplegable que integra el trabajo de decenas o cientos de desarrolladores.  
Para generar un desplegable, hay que partir de una línea de base (_baseline_) coherente. Al intentar trazar esta línea sobre una base de código donde contribuyen decenas o cientos de personas, es prácticamente inevitable que surjan conflictos, veamos algunos ejemplos.
- El código generado por un equipo A incluye una invocación de la forma  
`const a: number = area(3,8)`  
donde `area` es una función definida por otro equipo B. El equipo B modificó el nombre de la función (p.ej. a `getArea`), la cantidad y/o tipo de los parámetros (p.ej. cambió a un solo parámetro que es un objeto `{ width: 3, height: 8}`), y/o el tipo de retorno (ahora es un objeto `{area: 24, scale: 'small'}`), en cualquier caso sin que el equipo A se entere.
- el código incluye una operacion sobre una base de datos, utilizando una definición de esquema desactualizada, p.ej. hacer un `INSERT` sobre una tabla a la que se le agregó un atributo obligatorio que no está contemplado en el código.
- se repiten nombres de recursos, p.ej. dos equipos definen una tabla con el mismo nombre, o el mismo endpoint en un backend que expone una API HTTP.

A estos errores, que se detectan rápidamente al integrar la línea base candidata, se agregan otros más sutiles: debido a cambios en la lógica de un componente, p.ej. en cómo se calcula el resultado de una función o qué actualizaciones hace un proceso; los componentes que lo invocan simplemente empiezan a fallar, y para peor, tal vez sólo fallen en algunos casos. Esto implica la necesidad de realizar un testeo exhaustivo, y de más correcciones.

A su vez, un desplegable complejo también está expuesto a problemas de _despliegue_: podría ser necesario configurar una gran cantidad de parámetros, y es fácil que se escape alguno.

La suma de estos problemas puede provocar retrasos en la salida a producción de un proyecto de semanas, e incluso meses.

Otra consecuencia negativa del despliegue de un monolito, es que dificulta la posibilidad de _escalar un componente determinado_. P.ej. para un backend que expone una API extensa, si algunos endpoints en particular son muy usados, para replicarlos en varios nodos es necesario desplegar _todo_ el monolito en cada uno.


### Problemas sociales
Las dos cuestiones mencionadas son claramente _técnicas_. 
Por otro lado, el desarrollo basado en desplegables que integran el trabajo de muches devs, también conlleva riesgos relacionados con los aspectos _sociales_ del desarrollo de software, que muchas veces son tanto o más relevantes que los técnicos respecto del resultado de un proyecto.

Entre ellos destacamos la dificultad de _mantener un alto nivel de comunicación en un equipo extenso_, donde en principio cualquier equipo o desarrollador, puede utilizar un componente  de cualquier tamaño, generado por cualquiera de sus colegas.  
Otro problema es la _falta de motivación_ o de identificación con el proyecto por parte de los miembros del equipo, en un contexto donde sólo algunos pocos entre ellos pueden llegar a tener una visión de conjunto sobre el producto que se está desarrollando.  
También mencionamos las dificultades en el _manejo de los repositorios de código_.
Si se opta por tener un repositorio único, las contribuciones continuas de muchas personas van a atentar, necesariamente, contra la estabilidad de la base de código. Si se generan distintos repositorios, entonces se hace más complejo invocar desde el código que está generando un equipo, a una funcionalidad desarrollada por otro, y también se agrega complejidad al proceso de consolidación necesario para generar una versión.  
Finalmente, subrayamos que una gran base de código que deba mantener la coherencia necesaria para integrarse en un único desplegable, _atenta contra las posibilidades de experimentación o evolución_. Cualquier intento de aplicar una tecnología novedosa en un componente, podría causar incompatibilidades en otros componentes, o en la relación entre los mismos. 
Tales proyectos tienen un _alto riesgo de deuda técnica_, que es un factor adicional para la falta de motivación en sus integrantes.

Podemos resumir señalando que así como un monolito tiene problemas para escalar su operación, un equipo organizado para generar un desplegable único tiene problemas para escalar en tamaño. Se puede pensar en un desplegable único con un equipo de 6 personas que trabajan en la misma oficina, pero se hace enormemente más difícil con un equipo de 120 devs que están distribuidos en varias ubicaciones geográficas.


## El desarrollo basado en microservicios, ¿es beneficioso para cualquier proyecto?
La respuesta rápida a esta pregunta es "no". La idea de desarrollo basado en microservicios surge ante problemáticas en proyectos con ciertas características, y apunta específicamente a ese tipo de proyectos.

En nuestra opinión, un aspecto a considerar es la _escala_. En un proyecto que pueda llevarse a cabo con un equipo de 6 u 8 desarrolladores, con una base de código no muy grande, el trabajo adicional que requiere un desarrollo separado en distintos microservicios puede configurar un costo y/o riesgos mayores que los de mantener una única base de código y un solo entregable.

Otra cuestión a tener en cuenta es que en un despliegue que involucra decenas, o incluso cientos, de microservicios, una gran cantidad de _invocaciones_ que en un monolito serían llamadas dentro de un espacio de memoria, _se convierten en requests que viajan por una red_, lo que tiene un impacto considerable en la performance. En algunos productos, esta penalidad podría no ser tolerable.

También deben analizarse cuestiones de _consistencia_. En un sistema con gran cantidad de microservicios que colaboran entre sí, muchas de las transacciones de negocio involucran a varios microservicios, lo que impide manejarlas con el grado de aseguramiento en la consistencia que brinda una transacción resuelta en un único desplegable. 


### Variantes al "todo o nada"
Por otro lado, tampoco es cierto que _dentro de un mismo proyecto_, las únicas dos opciones posibles sean monolito único o desarrollo completamente basado en microservicios.

Una alternativa que se aplica en varios casos, en particular en el Banco del Sol, consiste en un core de negocio que es un monolito, que garantiza eficiencia y consistencia en las operaciones básicas, al que se suma una cantidad de microservicios que resuelven funcionalidades que son necesarias (pensemos p.ej. en el onboarding) pero menos centrales.

También, al menos en principio, resulta más sencillo aplicar las ideas de microservicios en el backend, en donde resulta fácil desplegar distintos componentes por separado, que en el frontend, sobre todo si se apunta a single-page applications.

Otro aspecto a considerar es que en aplicaciones _muy_ grandes, se puede llegar a un ecosistema con cientos, o incluso más de mil, microservicios. En un escenario de esta naturaleza, la complejidad que agrega la configuración y el monitoreo del ecosistema, puede ser mayor a la que se ahorra por la división en componentes de despliegue separado. Ver p.ej. [este artículo](http://highscalability.com/blog/2020/4/8/one-team-at-uber-is-moving-from-microservices-to-macroservic.html) donde describe una "vuelta atrás" parcial en Uber.

## Esta sección
En las páginas siguientes, vamos a describir algunas características del desarrollo basado en microservicios que juzgamos relevantes, y luego enunciaremos varias consecuencias en la organización del equipo y del desarrollo. 
Finalmente, veremos cómo algunas de las técnicas que trabajamos en etapas anteriores, resultan útiles en un contexto de desarrollo basado en microservicios.
