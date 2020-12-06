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

El otro objetivo que se busca con la división por funcionalidad, es una _mayor cercanía de les desarrolladorxs con el negocio_, que reduce el riesgo de errores por una deficiente comprensión de los requerimientos, y favorece que desde el equipo de desarrollo se puedan proponer mejoras o variantes.


### Equipos full-stack valoran desarrolladores full-stack
Finalmente destacamos que esta organización favorece la contratación de _desarrolladores full-stack_, que puedan resolver problemas de distintas características técnicas, y que tengan una orientación hacia resolver necesidades de la comunidad de usuarios.



## Proceso de cambio más ágil

porque no tenés que tratar con mil equipos.


## Se simplifican los tests de integración