---
layout: default
---

# El día a día en un entorno basado en microservicios
Como cierre de esta sección, indicamos algunas prácticas que aparecieron en etapas anteriores de esta actividad, que resultan relevantes para el trabajo en un entorno basado en microservicios.


## Interfaces de los servicios
En proyectos organizados como ecosistemas de microservicios, las interfaces juegan un rol crítico en el desarrollo. Por lo tanto, resulta muy conveniente definirlas y documentarlas correctamente.

### Definición de las interfaces
El mantener una _uniformidad_ en las definiciones de los endpoints dentro de un ecosistema, hace que resulte mucho más fácil comprender las interfaces, y desarrollar una intuición de cómo podría ser el endpoint que resuelve una determinada consulta o funcionalidad.  
En este sentido, seguir algunas pautas sugeridas por los [principios REST](../api-rest/rest), temática que abordamos en la Unidad 1, es una opción muy conveniente.

Un punto en particular es cómo cada microservicio informa, en las respuestas de sus endpoints, los _casos de error_.  
En algunos proyectos, se respetan los [códigos de error HTTP](https://www.restapitutorial.com/httpstatuscodes.html). En otros, se prefiere mandar siempre el código de "OK" (200 o 201 según la operación) y enviar la información sobre los errores en un atributo especial del payload.  
La decisión que se tome, _debe ser uniforme en todo el proyecto_.


### Identificadores
Otro aspecto a considerar es el de los _identificadores_ de las entidades que estén relacionadas con varios microservicios. Se puede usar un id generado por una base de datos, pero si la entidad está modelada (con distintos recortes de datos) en la BD de más de un servicio, ¿cuál elegir?  

Una opción es _designar un servicio_ (que puede ser ad-hoc) que se encargue de registrar la información relacionada con una cierta entidad, y que cualquier servicio que tenga que consultar o modificar esta información, deba hacerlo mediante pedidos al "servicio designado". En este caso, hay que evaluar el volumen tráfico que agregan estos requests.  

Una alternativa es utilizar un _identificador relacionado con el dominio_, como lo hicimos para los códigos de países en nuestro [primer ejemplo de servicio Nest](../nestjs-basics/inicio-app). 
Esta opción es particularmente conveniente para entidades que estén involucradas en la interacción entre nuestra aplicación y servicios externos.
Debe tenerse en cuenta que para muchas entidades, existen identificadores bien definidos: códigos ISO de países y monedas, CUIL/CUIT para personas físicas y jurídicas, CBU para cuentas bancarias argentinas, código SWIFT para bancos.

En algunos casos, existen varios códigos posibles. Por ejemplo, la ISO define dos códigos de países, uno de dos caracteres y otro de tres.  
En estas situaciones, es muy oportuno que se defina un código único, que sea utilizado de manera uniforme en el ecosistema de microservicios de una aplicación. En un ejemplo en el que se [combinan datos de distintas fuentes](nestjs-basics/distintas-fuentes) sobre un país, notamos el incordio que genera tener que invocar a servicios que mantienen distintos identificadores para una misma entidad.





### Documentación - Swagger y Postman
En un entorno donde los equipos son autónomos y pueden estar geográficamente distribuidos, resulta especialmente relevante contar con documentación adecuada de cada microservicio, que sea fácilmente disponible.  
Este es el vehículo principal para conocer las particularidades sobre cómo invocar a los servicios con los que se necesita interactuar cuando se desarrolla una determinada funcionalidad.

El formato más popular para documentar la API de un servicio es [Swagger](../swagger/swagger-intro), temática que hemos cubierto extensamente en la etapa 1. 
Como [ya mencionamos](../swagger/swagger-nestjs-empezamos), se puede generar automáticamente la documentación Swagger de un servicio a partir de decorators que se incluyen en los controllers y DTOs.

Para que esta documentación esté disponible para cualquier integrante en un proyecto, es conveniente dejar levantado un entorno del ecosistema de microservicios dedicado a la documentación, y eventualmente a la experimentación como describiremos luego.

Para acotar la necesidad de consultas para utilizar un servicio, la documentación Swagger merece cierta dedicación al completar los decorators. Entre los puntos a tener en cuenta, mencionamos los siguientes.
- Incluir descripciones textuales del propósito de cada endpoint.
- Usar [Markdown en las descripciones de Swagger](../swagger/swagger-markdown), para destacar puntos relevantes y formatear las descripciones apuntando a una mejor comprensión.
- Elegir ejemplos significativos para los atributos de los DTO, revisar cómo quedan los ejemplos de payloads de request y response, evaluar la conveniencia de generar ejemplos adicionales para endpoints críticos.
- Incluir los distintos códigos de error con una pequeña explicación.

En la visión global de un proyecto, debe tenerse en cuenta que los Swagger constituyen un asset valioso para mantener la calidad del código y simplificar la comunicación interna, por lo que resulta conveniente tener en cuenta la generación de esta documentación al asignar tiempos, e incluirla en las revisiones de código.

Otro formato que puede utilizarse para plasmar la API de un servicio son las _colecciones Postman_, que brindan a quienes pueden estar interesades en utilizar un servicio, ejemplos concretos de uso.  
Para potenciar este medio adicional de comunicación, se puede establecer algún repositorio en el que cada equipo pueda disponibilizar colecciones Postman con invocaciones de los servicios que mantiene, y configurar el entorno dedicado para documentación con los datos necesarios para que los endpoints de ejemplo se puedan ejecutar.



## Invocaciones entre microservicios
Mencionamos dos puntos a tener en cuenta cuando para resolver la funcionalidad de un servicio se requiera información que proveen otros servicios dentro de nuestro ecosistema. Este es en particular el caso de los _servicios BFF_, que reúnen y compaginan información de varios servicios del backend para simplificar la lógica de un frontend.

El primero es que, para mitigar la latencia al hacer varias invocaciones HTTP, podemos lanzar estas operaciones en paralelo utilizando [Promise.all](../async/promise-all), temática que trabajamos en la Unidad 0.

El segundo es que debe analizarse qué respuesta brindar si al invocar a varios servicios, algunos brindan la información esperada y otros _responden con códigos de error_. Este punto fue debatido al trabajar con un servicio que [combina datos de distintas fuentes](nestjs-basics/distintas-fuentes).  
De acuerdo a la criticidad de la información que brinda cada servicio, podemos dar como resultado de nuestro servicio consolidador: 
- o bien un código de error que refleje que no se cuenta con información suficiente para brindar una respuesta aceptable.
- o bien una respuesta incompleta, indicando cuál es la información faltante, lo que se puede hacer con valores particulares en los atributos correspondiente, o bien listando la información faltante en un atributo adicional.


## A modo de conclusión
Con esto concluimos nuestra _pequeña_ excursión a los desarrollos basados en microservicios.

Esta es una más de las iniciativas surgidas desde el ámbito del desarrollo, para lidiar con la **complejidad** del los proyectos de construcción de software.  
En nuestra actividad, debemos lidiar con dominios extensos, hardware complejo, interacción compleja entre componentes, grandes volúmenes de datos, requerimientos complejos en la interface de usuario, actualmente teniendo que considerar una variedad de dispositivos.  
En proyectos de gran envergadura (cientos de años-hombre o más), se suman dos factores adicionales de complejidad: la misma escala del proyecto, y el cansancio natural cuando un mismo desarrollo se prolonga en el tiempo.

Cuando varios de estos factores se reúnen en un mismo proyecto, el talento individual de sus integrantes, siendo una condición necesaria para el éxito, no resulta suficiente.  
Se requiere de conceptos y pautas de _organización_ que mantengan a un proyecto vivo, ágil, y cercano a las problemáticas que (se supone) debería resolver.  

La idea de desarrollo basado en microservicios es una de las respuestas que fue generando la misma industria ante repetidas historias de fracasos de proyectos en los cuales el peso de la estructura terminó hundiéndolos.  
Creo que es en este sentido en que conviene entenderla, y considerarla para que sea un factor de éxito en nuestras iniciativas: como una herramienta más en nuestra continua lucha contra la complejidad.






