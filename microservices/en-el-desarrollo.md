---
layout: default
---

# Microservicios - consecuencias en el desarrollo
En la [página anterior](./caracteristicas) reseñamos algunas de las principales características del desarrollo basado en microservicios. En esta, analizaremos brevemente el impacto de estas pautas en la organización y las prácticas del equipo de desarrollo.


## Equipos de trabajo descentralizados y autónomos
La descentralización en el producto (componentes separados y autónomos con interfaces bien definidas, evoluciones independientes y heterogeneidad tecnológica) se translada naturalmente al equipo del proyecto.  
Los desarrollos basados en microservicios fomentan la organización en equipos con un alto grado de autonomía en su trabajo, tanto en las tecnologías que utiliza (como ya mencionamos) como en su forma de organización.  
Esta autonomía puede estar mediada por la existencia de estándares que deban ser respetados por todos los participantes de un proyecto. Pero estos estándares son, al menos en principio, pautas a respetar o guías que ayudan en la organización, y no estructuras rígidas de cumplimiento obligatorio; cada equipo tiene un amplio grado de libertad para definir su forma de trabajo.  

A su vez, la organización en equipos autónomos fomenta la _descentralización_ del  desarrollo, resultando menos relevante la reunión en un mismo lugar físico de todo el equipo que participa en un proyecto. Incluso pueden formarse equipos que colaboran en un mismo proyecto, en distintas ubicaciones geográficas.

Otra consecuencia de esta forma de organización dentro de un proyecto, es que al relacionarse cada integrante del proyecto con una cantidad acotada de colegas (los integrantes de su mismo equipo), se facilita la modalidad de _trabajo remoto_.

Destacamos finalmente que un requisito para que esta estrategia de organización de un proyecto sea exitosa, es que se incluyan integrantes con el seniority necesario en cada equipo que se conforme.


## Equipos full-stack, división por funcionalidades
Una de las pautas del desarrollo basado en microservicios, sugiere que cada equipo sea responsable no tanto por un conjunto de microservicios, sino por la _funcionalidad de negocio_ que los mismos resuelven.  
Por lo tanto, cada equipo integra responsabilidades sobre modelo y funcionamiento de la base de datos, implementación de la lógica de negocio, comunicación con otros microservicios del mismo ecosistema o servicios externos, y en algunos casos, también la UI. Es decir, _cada equipo es full-stack_. 

En esta propuesta, la organización del proyecto es _por funcionalidad_. Esto se contrapone a la visión tradicional de organización por especialización, en la que encontramos un equipo de base de datos, otro de lógica de negocios, otro separado de comunicaciones, y finalmente un equipo dedicado a la UI (o varios, si deben contemplarse distintos tipos de dispositivos o canales).  
Llevando esta tendencia al extremo, se puede asignar al equipo encargado de una funcionalidad, la responsabilidad de realizar las tareas asociadas de operaciones y monitoreo. Tal como se describe en [esta entrevista](https://queue.acm.org/detail.cfm?id=1142065), esta idea se implementa en Amazon bajo la consigna "you build it, you run it".

En proyectos complejos, los equipos full-stack pueden _complementarse con equipos especializados_ en tecnologías o tareas específicas, que puedan brindar consultoría y asistencia a los equipos operacionales.  
Muchas veces, dado el alto seniority que requieren estas tareas de consultoría y apoyo, este rol es cubierto por _equipos de arquitectura_ que se encargan de definir algunas pautas que ayuden al despliegue y monitoreo de los microservicios, y/o de construir algunas librerías que resuelvan tareas específicas requeridas en distintos servicios.


### Objetivos de este enfoque
Mencionamos dos de los objetivos que se buscan con esta propuesta de organización de equipos de proyecto.

Uno es que fomenta la _identificación_ de las personas que trabajan en un proyecto con los resultados de su trabajo. Formar parte de un equipo de dimensiones acotadas que tiene una responsabilidad clara respecto de la funcionalidad, tiende a generar un mayor vínculo de cada persona con lo que produce, y a partir de este vínculo, un mayor grado tanto de satisfacción como de compromiso. Aleja las sensaciones de alienación y ajenidad asociadas a los equipos grandes de trabajo en los que las responsabilidades están menos definidas.  
En este sentido, las prácticas de división por funcionalidades y de autonomía de cada equipo se relacionan con otras ideas como [Collective Code Ownership](https://www.agilealliance.org/glossary/collective-ownership).

El otro objetivo que se busca con la división por funcionalidad, es una _mayor cercanía de les desarrolladorxs con el negocio_, que reduce el riesgo de errores por una deficiente comprensión de los requerimientos, y favorece que desde el equipo de desarrollo se puedan proponer mejoras o variantes.


### Equipos full-stack valoran desarrolladores full-stack
Finalmente destacamos que esta organización favorece la contratación de _desarrolladores full-stack_, que puedan resolver problemas de distintas características técnicas, y que tengan una orientación hacia resolver necesidades de la comunidad de usuarios.



## Proceso de cambio más ágil
Suponiendo un escenario de equipo organizado por tecnologías o tareas, y no por funcionalidades de negocio, pensemos brevemente cómo podria reaccionar ante una necesidad de adaptar o ampliar una funcionalidad existente.  
- Después de un análisis exhaustivo por parte de analistas de negocio, el pedido de cambio debe pasar por una instancia de coordinación del desarrollo, que desglose las tareas correspondientes a cada equipo: las modificaciones necesarias en la estructura de la base ( las bases) de datos, los agregados o cambios a implementar en la lógica de negocio y en los servicios de datos que brinda el backend, y los ajustes que se requieren en la UI. Esto puede requerir de consultas previas con los equipos encargados de cada tarea, para obtener un alcance preciso de los cambios necesarios.  
- El resultado de esta etapa es un conjunto de tareas a ser asignadas a cada equipo, que probablemente deban ejecutarse en cascada: primero los cambios en las bases de datos, luego las modificaciones en la lógica de negocio, y finalmente las adaptaciones de la UI.  
- Si alguno de los equipos descubre alguna inconsistencia, p.ej. si el equipo de lógica de negocio descubre mientras está implementando que se requiere de un atributo no previsto en la base de datos, esto requiere de una coordinación entre equipos dedicados a distintas tareas, que eventualmente debe estar mediada por la instancia de coordinación.
- Cuando se concluye con las tareas planificadas, se realiza un testeo de las modificaciones. Si se encuentran defectos, debe dilucidarse cuál/es de los equipos de tareas deben realizar los ajustes necesarios, y definirse nuevas tareas para los mismos.

Este proceso ha provocado, en muchos proyectos, que la implementación de cambios requeridos desde el negocio lleve tiempos enormes, incluso para modificaciones sencillas.  
Se corre el grave riesgo de que un proyecto con este patrón de comportamiento se perciba como _anquilosado_, al igual que cualquier organismo que tiene tiempos demasiado prolongados para adaptarse a los cambios en su entorno.  
La consecuencia es una distancia creciente entre la funcionalidad construida y la que realmente necesita el negocio, lo que puede llevar a una obsolescencia temprana.

Una organización del equipo por funcionalidades permite un tratamiento mucho más ágil de los pedidos de modificaciones, adaptaciones y/o agregados que atañen a una funcionalidad, dado que la responsabilidad por los mismos recae en un único equipo, que además está familiarizado con el aspecto de negocio al que aplican los cambios pedidos. 
Esto evita la necesidad de una planificación detallada de los cambios a realizar, mitiga mucho la coordinación requerida entre distintos equipos, y agiliza el tratamiento de eventuales defectos detectados en el testeo.

Esta es, tal vez, la fortaleza más relevante de la organización por funcionalidades.


## Se favorece la experimentación
La combinación de componentes desplegables por separado y con evolución independiente, heterogeneidad de tecnologías, y equipos con alto grado de autonomía, genera un escenario propicio para la experimentación con nuevas tecnologías, lenguajes, librerías, o técnicas de codificación.

Se puede elegir un microservicio para llevar a cabo una _implementación experimental_ aplicando la faceta que se quiera probar, en esta tarea pueden colaborar el equipo responsable del microservicio con un equipo de arquitectura o consultoría/revisión de código.  
- Los tests de integración dan una primera medida del comportamiento de la implementación alternativa. De existir tests que midan la performance u otros atributos arquitecturales, pueden suministrar información adicional.  
- Esta implementación puede ser deplegada en un entorno operativo con un número de versión diferenciado, para poder comprobar su funcionamiento en producción. Inclusive, se pueden desplegar (también con números de versión diferenciados) otros componentes que interactúen con el que se está probando, para comprobar la interacción.
- Finalmente, se puede dejar en firme la nueva versión, a la que se acoplarán los otros microservicios a medida que reconfiguren la versión a utilizar. 

Notamos que el _uso de interfaces sencillas_ entre componentes, facilita en gran medida la implementación y puesta en producción de una nueva versión de un microservicio, implementada aplicando elementos novedosos. En rigor, los detalles de implementación de cada componente no tienen influencia en su rol dentro del ecosistema de microservicios, mientras se respete la interfaz.

Subrayamos que también en el aspecto técnico, el desarrollo basado en microservicios mitiga el riesgo de anquilosamiento de un proyecto.


## Se simplifica el testing
Finalmente, señalamos que la separación de una gran masa de código en microservicios, sumada a la definición de interfaces sencillas, simplifica la creación y ejecución de tests.

Por un lado, al quedar cada componente "físicamente" separado del resto, está claro cuál es el alcance de los tests internos al componente, y resulta natural ejecutar esos tests sin interferencias del resto del proyecto.  
Para los tests que incorporen la interacción de un microservicio con otros, las interfaces sencillas facilitan la generación de mocks de los servicios a los que se invoca.

Una alternativa que puede evaluarse es que cada microservicio (o al menos, los microservicios más "populares" en términos de otros servicios que les hacen pedidos) publique, en un entorno de desarrollo, una "versión mock" con comportamiento bien definido y acotado, para el uso en tests de los microservicios que lo invoquen.

