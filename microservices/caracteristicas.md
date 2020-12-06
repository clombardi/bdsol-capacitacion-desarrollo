---
layout: default
---

# Microservicios - algunas características
En esta página, vamos a repasar algunas de las características de los desarrollos basados en microservicios.  
Veremos que aunque estas características se describan por separado, se complementan formando un escenario coherente para el desarrollo de aplicaciones de mediana o gran escala. También mencionaremos brevemente algunas relaciones del desarrollo basado en microservicios con otras iniciativas dentro del ámbito IT.

## Varios desplegables, que se comunican entre sí con interfaces sencillas
La pauta inicial para el desarrollo basado en microservicios es que 
- en lugar de tener _un monolito_, es decir, un único producto desplegable que incluye a todas las funcionalidades, 
- se generan **varios desplegables**, que interactúan entre ellos mediante servicios con interfaces sencillas.

De esta forma, se evitan los problemas mencionados en [las motivaciones](./intro-movida), asociados a la generación y despliegue de un monolito.  
Por otra parte, esta práctica habilita, o al menos facilita, varias de las restantes pautas asociadas al desarrollo basado en microservicios; es el punto inicial para configurar una estructura distinta del desarrollo.

Las _interfaces_ de los servicios se implementan, por lo general, como API HTTP. Para _los datos que deban "viajar"_ entre servicios, se utilizan lenguajes de intercambio de información, como JSON, XML o YAML. Esto se contrapone a la idea de serialización de objetos o funciones; entre los componentes sólo se intercambian datos planos.

Un aspecto que debe considerarse es que, al tiempo que se simplifica la generación y despliegue de cada servicio individual, se genera un entorno de _ejecución más complejo_, en el que deben convivir varios desplegables, deben configurarse los accesos entre los mismos, y debe procurarse máxima eficiencia para los requests que se utilizan para la comunicación entre servicios.   
El surgimiento de entornos de estas características fomentó la aparición de herramientas que simplifican el despliegue de múltiples productos, tales como Docker o Kubernetes. A su vez, los entornos cloud resultan adecuados para este tipo de arquitecturas, pues facilitan la replicación a demanda de múltiples servicios.  
Estos son ejemplos de cómo las ideas del desarrollo basado en microservicios, guían la evolución del ámbito de operaciones IT.

Uno de los desafíos, y a su vez de los riesgos, de esta pauta es la _necesidad de definir los componentes_ en los que se va a distribuir la funcionalidad de una aplicacion, y demarcar las responsabilidades de cada uno. Una organización incorrecta de la funcionalidad en el conjunto de microservicios, puede provocar un tráfico excesivo de mensajes entre los mismos, y/o la repetición de lógica en varios microservicios que implementan distinas facetas de una misma funcionalidad.


## Despliegues más frecuentes
El más evidente entre los efectos buscados al "romper el monolito" generando distintos desplegables, es simplificar la generación y despliegue de los resultados del desarrollo.

El desarrollo basado en microservicios propone aprovechar fuertemente esta posibilidad, promoviendo la práctica de desplegar en forma frecuente, nuevas versiones de los microservicios. Dicho de otra forma, que el equipo de desarrollo y operaciones "le pierda el miedo" a generar y desplegar nuevas versiones.  
Esta práctica permite reaccionar rápidamente ante la aparición de errores, o la necesidad de agregar funcionalidad en forma ágil.  
_Llevada al extremo_, esta pauta sugiere hacer del despliegue de componentes una operación rutinaria, subiendo nuevas versiones en forma semanal o incluso diaria.  

Para poder llevar a cabo esta idea, y en particular para que represente un beneficio real para el desarrollo en lugar de una carga adicional para los equipos de desarrollo y operaciones, la separación en distintos entregables es un paso necesario pero no suficiente.
Debe generarse una estructura que realmente haga más sencillo el proceso que va desde que el código candidato para una versión de un componente se considera listo, hasta que el componente está operativo. Entre otros, hay que contemplar los siguientes aspectos
- definir una _política de branching_ adecuada a las necesidades del proyecto, y documentarla/comunicarla correctamente.
- contar con baterías extensivas de _tests automáticos_, tanto unitarios como de integración.
- _automatizar procesos_ de generación y despliegue.

Esta pauta vincula al desarrollo basado en microservicios con la iniciativa popularizada como _DevOps_, que consiste en acercar los ámbitos de desarrollo y operaciones, y aplicar técnicas y herramientas de desarrollo en los procesos de operaciones.


## Evolución independiente
En un entorno de componentes que se despliegan por separado y en forma frecuente, tener que sincronizar el despliegue de nuevas versiones de todos los componentes produce efectos contrarios a los deseados: se pierde la rapidez en la reacción, y se agrega complejidad al proceso de despliegue al requerir que todos los componentes se actualicen al mismo tiempo.

Por lo tanto, en la lógica del desarrollo basado en microservicios, se asume que los mismos tienen _historias independientes_. 
El despliegue de una nueva versión de un componente no implica, al menos en principio, 
En un determinado momento, podrían coexistir en un mismo proyecto, componentes que hayan pasado por varias versiones (por falta de precisión en la comprensión inicial del problema, requerimientos agregados o mejoras técnicas, entre otros factores) con otros más estables.

Al mismo tiempo que quita presiones y agrega agilidad al proceso de desarrollo, la evolución independiente de cada microservicio genera un nuevo requerimiento sobre los entornos operativos: la necesidad de manejar la historia de _versiones de cada componente_. 
En particular, puede resultar necesario tener operativas, en un mismo momento, distintas versiones de un mismo componente.  
Para describir un ejemplo, supongamos un entorno con cuatro componentes A, B, C y D, donde los componentes B, C y D le hacen pedidos al componente A. En un momento determinado, el componente A tiene que cambiar su interface, tal vez de forma no retrocompatible, por una necesidad vinculada con la funcionalidad que resuelve el componente D. Digamos que el componente A pasa de la versión 1.2.0 a la 2.0.0.
Se deberá desplegar la versión 2.0.0 de A, junto con una versión actualizada de D que utilice esta nueva versión de A.  
Para mantener la idea de evolución independiente, los componentes B y C se actualizarán a la versión 2.x de A cuando necesiten sacar una nueva versión por razones intrínsecas, y no antes. Por lo tanto, hasta que B y C (y en general, todos los clientes de A) se adapten a la versión 2.x de A, la versión 1.2.0 tiene que seguir estando operativa.

En este escenario, todos los pedidos a un microservicio deben especificar la versión (o rango de versiones, p.ej. usando [SemVer](https://semver.org/)). A su vez, para redireccionar los pedidos de acuerdo a la versión, se puede implementar un punto de entrada, p.ej. en un container adicional en el mismo pod en el que se despliegan las distintas versiones.


## Ecosistemas heterogéneos
Otra consecuencia de la decisión de "romper el monolito" y pasar a un ecosistema de componentes, es que también se "rompe" la necesidad de mantener la homogeneidad en un proyecto, o sea, de manejar un único lenguaje de programación, stack tecnológico, o motor de BD.  
En principio, el equipo responsable del desarrollo de cada microservicio puede realizar las elecciones tecnológicas que le resulten adecuadas de acuerdo a las condiciones específicas del mismo, entre otras:
- necesidad de conectarse con servicios externos que hacen conveniente utilizar cierta librería, que está disponible sólo para algunos lenguajes de programación.
- organización de los datos que hay que persistir, que puede influir en el tipo de base de datos (SQL, de documentos, orientada a grafos, etc.) a utilizar.
- requerimientos particulares de performance.
- el background de los integrantes del equipo, y/o la facilidad de sumar integrantes adicionales de ser necesario.

Desde el punto de vista de la interacción, el uso de interfaces livianas hace que las decisiones tecnológicas de cada componente sean transparentes. Queda claro que hay un aspecto en que resulta altamente conveniente mantener homogeneidad, que es el mecanismo de comunicación entre microservicios. P.ej. es incómodo que en un mismo proyecto coexistan  servicios que usan JSON para los payloads de requests y responses, con otros que usan XML.  

Por otro lado, una heterogeneidad excesiva en un proyecto puede traer problemas en el ciclo de desarrollo.
Mencionamos dos de estos riesgos: puede resultar complicado realizar revisiones de código por parte de miembros externos al equipo, p.ej. de un equipo de arquitectura o de consultoría, y puede dificultarse la integración en un equipo, de un integrante del proyecto proveniente de otro equipo en el que se usan tecnologías muy distintas.

Un camino intermedio es la definición, por parte de un equipo de arquitectura, de varias opciones de stack tecnológico que pueden utilizarse en el proyecto, de forma tal de permitir que cada equipo elija la variante más adecuada a sus necesidades y background, sin que esto atente contra la mantenibilidad.


## Datos descentralizados
Para cerrar esta página, señalemos que un ecosistema formado por microservicios con un alto grado de autonomía, fomenta la descentralización de las bases de datos. 

Esta situación evita un riesgo que está presente en muchos proyectos donde los datos están centralizados: que la base de datos sea el cuello de botella para la performance.  
Por otro lado, se presenta un desafío para el trabajo de operaciones: el de generar copias de resguardo y mecanismos de recuperación ante fallos para datos que están distribuidos en una multitud de bases, que pueden tener características distintas.

En principio, cada microservicio maneja su propia base de datos. También puede organizarse un proyecto de forma tal que el manejo de ciertos datos críticos esté centralizado en un único microservicio "responsable" de los mismos, y que los servicios que requieran obtener y/o actualizar estos datos, tengan que recurrir al "responsable" enviándole los pedidos necesarios.
