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
La decisión que se tome, debe ser uniforme en todo el proyecto.


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

Promise.all

Errores
## Final word
Complejidad.
