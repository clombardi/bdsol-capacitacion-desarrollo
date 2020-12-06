---
layout: default
---

# Microservicios - algunas características
En esta página, vamos a repasar algunas de las características de los desarrollos basados en microservicios.


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

