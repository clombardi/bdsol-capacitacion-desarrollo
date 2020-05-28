## Un poco de contexto
Hasta ahora trabajamos con conceptos bastante genéricos: JS / TS, operaciones externas, procesamiento asincrónico. 
Todas estas herramientas se aplican en proyectos de desarrollo de distintas características. 

En lo que viene, vamos a acercarnos a lo "concreto". 
Nos vamos a concentrar en el desarrollo del backend de un software organizacional, con dos características relevantes
1. la interface es mayormente API REST por HTTP
1. se aplican ideas del desarrollo basado en microservicios.

Cada microservicio atiende uno o varios _endpoints_, para cada invocación le llega un _request_ que puede incluir parámetros de path y/o de query, body, y/o headers. Hay que atender cada uno de estos endpoints, generando la _response_ que corresponde.  
Todos los datos relevantes, incluyendo p.ej. datos de autenticación, vienen en el request.  
Algunos de estos endpoints son _consultas_, otros son _operaciones_  que modifican algún estado (saldos, movimientos, pagos, datos de cliente, etc.).

Llamemos _request handler_ al código que responde a un endpoint. Un request handler puede pensarse como una función, que recibe un request y devuelve la response.  
Para llevar a cabo su cometido, un handler puede invocar a otros microservicios, acceder a bases de datos, interactuar con sistemas externos, entre otras acciones.  
También es posible que deba realizar validaciones sobre datos del request. 


## NestJS - qué es
Se auto-define como un framework para construir backends que se ejecutan en Node.  
NestJS **no** provee la funcionalidad básica de server HTTP, utiliza una librería de server, por defecto (y en BDSol) es [ExpressJS](https://expressjs.com/).

Entonces, si no provee el server, ¿qué provee NestJS?  
Varias cosas de las cuales ya están usando algunas, que iremos repasando, y tal vez hasta sumándole otras, durante esta capacitación.

En principio, provee una _organización_ para el código de los request handlers.  
Tenemos _módulos_, que se pueden ensamblar entre ellos para armar un servicio.
Cada módulo incluye _controllers_, que definen los handlers indicando qué rutas atiende cda uno, y _providers_ (entre ellos los servicios) que implementan las acciones necesarias para resolver cada request.  

En NestJS, un request handler es un método en un controller que tiene un decorator correspondiente al verbo (`@Get`, `@Post`, etc.). 
El framework provee algunas facilidades que simplifican el desarrollo de request handlers.  
En particular, en muchos casos no es necesario manipular los objetos `request` y `response`, porque permite acceder a los datos que se usan más frecuentemente definiendo parámetros del método, y genera el response a partir del valor de respuesta.  
Además provee un mecanismo para definir las rutas usando los decorators, sin necesidad de escribir código específico; y otro para manejar los casos de error a partir de excepciones, generando los response correspondientes.

NestJS le da vuelo al concepto de **middleware**, que ya aparece en Express (y también en alternativas anteriores como p.ej. SpringBoot). 
Se definen distintas variantes de middleware para propósitos específicos.

Finalmente, mencionamos que NestJS provee conectores que simplifican el acceso a bases de datos mediante ORM (incluyendo Mongoose, TypeORM y Sequelize), y también soporte para distintos tipos de test.


## Qué vamos a hacer ahora
El objetivo es ir completando y enriqueciendo el mapa mental de NestJS que cada une de ustedes ya tiene, dado que ya lo vienen usando.  
También, que lo relacionen con varios de los temas dados al principio, en particular: interfaces, tipos, generics, promesas y async/await.

Vamos a empezar implementando un servicio que accede y consolida datos de sitios externos, los mismos que usamos en el ejercicio sobre procesamiento asincrónico.  
Aprovecharemos para repasar cómo maneja Nest la definición de rutas, un poco de manejo de errores, y algunos decorators.

Intentaré no repetir en estas páginas la info que ya está en [la documentación de NestJS](https://docs.nestjs.com/).