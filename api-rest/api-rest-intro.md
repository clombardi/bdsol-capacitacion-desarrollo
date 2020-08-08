---
layout: default
---

# API REST - interface de un microservicio
Vamos a poner nuestra atención en algunos conceptos relacionados con la _interface_ que **expone** un microservicio.   
Al destacar la palabra _expone_, queremos subrayar que la interface es lo _único_ que otros componentes de un sistema pueden conocer de un microservicio: la forma de comunicarse.

Hablaremos sobre [**REST**](https://restfulapi.net/), que no es otra cosa que un conjunto de principios que guían la definición de una interface, junto con las metas que dieron lugar a definir esos objetivos.  
Es interesante conocer un poco sobre las ideas de REST, porque contribuyen a darle impulso a la idea de microservicios, y a que la interface de los microservicios tenga la forma que conocemos. Una interface basada en el protocolo HTTP y en el uso de un formato de intercambio de información liviano, como es JSON, forman una buena plataforma tecnológica para cumplir con lo que propone REST. 

Después, estudiaremos algunas particularidades del protocolo HTTP, y debatiremos sobre cómo definir los endpoints que forman la interface de un microservicio, de acuerdo con las ideas que propone REST.

Finalmente, mencionamos que trabajar con estos temas nos va a dar la oportunidad para empezar a pensar en la _arquitectura_ de una aplicación organizacional.
