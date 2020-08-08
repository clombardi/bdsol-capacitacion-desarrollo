---
layout: default
---

# Ejercicio integrador
Hasta ahora no hemos planteado ningún desafío referido a la temática de API REST. 
Preferimos plantear un ejercicio en el que se puedan aplicar distintas ideas que aparecieron en este contenido, para llegar a la definición de una API compleja.

Por ahora vamos a hacer esto con papel, lápiz y paciencia. Más adelante, veremos cómo definir APIs usando [Swagger](https://swagger.io/).


## API de un microservicio de solicitudes de cuenta
En concreto, les proponemos que definan la API completa para una versión de un microservicio de solicitudes de cuenta que debe cubrir la funcionalidad que se detalla.
Se deben indicar qué endpoints hay que incluir, y para cada endpoint
- qué query params, path params y/o headers específicos deben contemplarse
- body del request, si hay body
- body de la respuesta
- status code posibles.

En todos los casos, además de los datos detallados, agregar los identificadores que sean necesarios.

**Oficiales del banco**  
De cada oficial hay que registrar el nombre y la fecha de ingreso al banco.
Se debe poder hacer un CRUD completo de oficiales.
- alta
- eliminación
- modificación completa de datos.
- consulta de todos los oficiales.
- consulta de los datos de un oficial por id.
- consulta de los oficiales que hayan ingresado antes de una fecha determinada, después de una fecha determinada, o entre dos fechas.

**Solicitudes de cuenta**  
De cada solicitud se debe registrar: fecha, cantidad de aprobaciones necesarias, cliente (nombre y apellido), estado que puede ser: en análisis, aprobada, rechazada, ejecutada, archivada.
Se debe poder hacer 
- alta de solicitudes, el estado inicial es "en análisis".
- modificación de nombre del cliente.
- consulta de solicitudes, filtrando por estado y/o nombre del cliente, ordenando por nombre del cliente o por fecha.

**Aprobación de solicitud de cuenta**  
Es el acto por el cual un oficial aprueba una solicitud.
Se indica: oficial que hizo la aprobación, fecha, solicitud.
Se debe poder:
- registrar una aprobación.
- consultar las aprobaciones para una solicitud.
- consultar las aprobaciones hechas por un oficial.
- en la consulta de solicitudes, dar la posibilidad de incluir o no la cantidad de aprobaciones hecha para cada una.

**Archivado de solicitudes**  
Es un proceso por el cual todas las solicitudes en análisis anteriores a cierta fecha pasan al estado "archivada".
Se debe indicar la fecha en cuestión.

**Incremento de cantidad de aprobaciones necesarias**  
Se debe agregar una operación que le sume uno a la cantidad de aprobaciones necesarias para una determinada solicitud. Sólo tiene sentido para solicitudes en análisis.
