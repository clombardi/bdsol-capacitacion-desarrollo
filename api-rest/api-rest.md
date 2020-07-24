# API REST (finalmente)
En la [página anterior](./rest.md), describimos los principios que propone REST.  
Ahora, vamos a estudiar qué propiedades debe cumplir la _interface_ de nuestros servicios, para cumplir con estos principios.

## Qué nos brinda la arquitectura elegida
Analicemos los principios REST en el contexto de una arquitectura como la definida en Banco del Sol, que refleja estas características.
- el backend está basado en microservicios.
- el frontend principal es una UI Web.
- la comunicación entre microservicios, y entre backend y frontend, se resuelve mediante pedidos HTTP.
- la información en pedidos y respuestas se representa en JSON.

Estas definiciones iniciales facilitan la observancia de los principios de _arquitectura cliente-servidor_ y _por niveles (layered)_.

Respecto de _cacheable_, hay varios elementos que hacen simplifican su implementación:
- la representación uniforme de los datos en requests y responses (i.e. el formato JSON). 
- como ya lo indicamos, una arquitectura por niveles habilita la aparición de componentes intermedios, que pueden incluir caches. 
- el protocolo HTTP incluye soporte para la declaración de qué datos son cacheables, lo que mencionaremos brevemente un poco más adelante.

Abordaremos con más profunidad el establecimiento de caches en microservicios en una etapa posterior de esta capacitación.

Por lo tanto, para que la  **interface** de nuestros servicios sea _RESTful_ (o sea, cumpla con los principios REST) nos focalizamos en los principios de _comunicación stateless_ e _interface uniforme_, recordando sobre este último que REST recomienda basar la identificación de cada endpoint en el _recurso_ afectado.


## API HTTP
El término _API_ se refiere, simplemente, a la interface que presenta una pieza de software (literalmente es "Application Programming Interface"). Dejamos una [pequeña nota histórica](https://nordicapis.com/who-invented-the-api/) para les curioses.

En la arquitectura que describimos, la API de un microservicio está formada por un conjunto de _endpoints HTTP_, donde un endpoint se define como URL + método. 
P.ej. `GET /account-requests` y `POST /account-requests` son dos endpoints distintos, que representan distintas _acciones_ sobre un mismo _recurso_.

> **Recordemos**  
> Un endpoint se corresponde con un request handler en NestJS.


### Parámetros HTTP
El protocolo HTTP define distintos tipos de _parámetros_, o sea, información adicional asociada a un pedido. En particular, tenemos
- query parameters
- headers
- body
- cookies

En la definición de APIs, se suman los llamados _path parameters_, que desde el punto de vista de HTTP son parte de la URL, pero desde la perspectiva de la API son un tipo de parámetro más. 

> **Nota**  
> Podría plantearse el debate "filosófico" sobre si `/account-requests/id-1` y `/account-requests/id-2` son dos endpoints distintos, dado que se refieren a dos recursos distintos, o el mismo con un parámetro.  
> En este material adoptamos una perspectiva pragmática, en la que "vemos" un único endpoint `account-requests/:id`.


## Cómo (no) ser stateless
Para garantizar que nuestra API sea stateless, debemos verificar que el comportamiento del servicio al recibir un pedido dependa únicamente de la información incluida en el pedido, y no tenga relación con pedidos anteriores.

Por ejemplo, un protocolo para agregar una solicitud de cuenta de esta forma
1. avisar que se va a querer crear una solicitud de cuenta.
1. enviar el nombre del cliente.
1. enviar la cantidad de aprobaciones requerida.
1. indicar "fin-de-datos" para que se realice el alta.

no es stateless, porque los datos de la nueva solicitud, que se genera en el último pedido, dependen de los datos enviados en pedidos anteriores. 

Pensando en _porqué_ conviene no romper el principio de comunicación stateless, notemos que el protocolo definido exige que _el server_ mantenga información sobre cada operación "abierta", asociándola p.ej. a algún identificador de sesión de cliente.  

Además, supongamos un servicio que viene corriendo en una única instancia, y al que se decide agregar varias instancias con un load-balancer.
En el nuevo escenario, para que el protocolo definido funcione, hay que garantizar que todos los pedidos que forman una misma operación sean atendidos por el mismo servidor, pensar qué pasa si ese servidor se cae en el medio de la operación, y tal vez otras complicaciones. 

En resumen, un protocolo como el planteado aumenta la complejidad del server. Notemos también que se degrada la performance: hacer cuatro pedidos implica usar más recursos que hacer uno.   
Y no estamos agregando valor a la funcionalidad que brinda el servicio: usando este protocolo, o definiendo un único endpoint `POST \account-requests` donde se pasan los datos en el body, estamos brindando _la misma_ funcionalidad: agregar una nueva solicitud de cuenta.

**Conclusión**:  
mantener una interface stateless nos orienta hacia una mayor _simplicidad_ en los distintos componentes que forman una aplicación.


### Otro ejemplo
Otro ejemplo de una interface no stateless sería un endpoint `PUT /account-requests` donde se envían los nuevos datos en el body, pero no se envía el id, confiando en que el servidor va a aplicar los cambios a la última solicitud agregada.


### ¿Y si se _necesita_ hacer una operación por pasos?
Existen casos en los cuales una operación _debe_ hacerse por pasos. Dos motivaciones usuales son:
- el requerimiento de soportar _cargas parciales_ por parte del usuario, o sea cargar los datos de la nueva entidad paulatinamente en distintas sesiones.
- el _volumen_ de los datos asociados hace no recomendable enviarlos todos en un mismo request.

Si se presenta esta situación ¿se puede resolver sin perder el "sello RESTful"?

La respuesta es (creo) **sí**.  
Una palabra clave al respecto es "modelar", debemos incorporar esta situación al _modelo_ que maneja nuestro servicio.

Hay una nueva entidad que representa el alta de una nueva solicitud, que va a tener su ciclo de vida. Podemos definir un endpoint `POST /account-request-creation` para crear una nueva alta, el resultado es un id que representa _a la operación de alta_.  
Para agregar datos a esta alta, podemos definir `PATCH /account-request-creation/:id`. 
Si el usuario decide cancelar el alta, usaremos `DELETE /account-request-creation/:id`. 

¿Qué hacemos cuando el usuario decide _confirmar_ el alta, o sea, crear la nueva solicitud?  
Podemos _modelarlo_ como un proceso: `POST /account-request-creation-fullfillment/:id`.  
El proceso consiste en la transformación de un "alta de" en una nueva solicitud. El resultado es el identificador de la nueva _solicitud_, la operación de alta deja de estar activa. 

En esta propuesta, la complejidad que agrega el hecho de tener que soportar una operación "por pasos", se refleja en el modelo, no queda "escondida" en un protocolo.  
Además, se obtienen beneficios adicionales.
- un mismo usuario puede tener varios procesos del mismo tipo abiertos, porque cada uno tiene su identificador específico.
- agregando un endpoint `GET /account-request-creation/:id`, se pueden obtener los datos cargados hasta el momento.
- el registro de las operaciones de creación puede mantenerse, para obtener estadísticas sobre el comportamiento de los usuarios (qué datos suelen cargar en una misma sesión, cuántas sesiones ocupa una operación de creación) que permitan pensar mejoras en la interface.


## Interface uniforme basada en recursos
Sobre este tema en particular, hay una plétora (una forma elegante de decir "un montón") de tutoriales en Internet, [este](https://restfulapi.net/rest-api-design-tutorial-with-example/) me pareció prolijo.  
Nos limitaremos aquí a dar algunas indicaciones.

En principio, en cada endpoint, el _path_ de la URL debe identificar al recurso al que se refiere el pedido.

Para dar un ejemplo sencillo, un endpoint como 
```
GET /get?type=account-requests
```
es incorrecto, porque el recurso (en este caso, la colección de solicitudes de cuenta) está en un query param. El endpoint correcto es
```
GET /account-requests
```

Por otro lado, la _acción_ a realizar no debe formar parte de la URL, sino que debe ser especificada mediante el método. Por lo tanto, un endpoint
```
POST /create-account-requests
```
no es correcto, porque la acción está descripta en el método, que es POST, no es necesario, ni recomendarlo, que forme parte de la URL. 
Además, evitamos posibles incoherencias como
```
PUT /create-account-requests
```

> **Nota**  
> Se recomienda que los nombres de las colecciones vayan siempre en _plural_, o sea, vamos a preferir p.ej. `GET /account-requests/:id` por sobre `GET /account-request/:id`.



### Recursos anidados
Pasemos por un momento a un dominio clásico: un modelo de personas. Para acceder a la información de una persona, se usa este endpoint.
```
GET /persons/:id
```

Una persona puede tener información asociada, p.ej. (otra vez, típicamente) las direcciones. Esto define un nuevo recurso "la colección de direcciones de una persona". Este es un caso de _recurso anidado_: este recurso se puede pensar como que está "dentro" de la persona.  
Para estos casos, se recomienda usar una URL con este formato
```
/persons/:id/addresses
```

Esto nos permite especificar la búsqueda de las direcciones de una persona
```
GET /persons/:id/addresses
```
y también agregar en forma sencilla una dirección nueva, que es para una determinada persona
```
POST /persons/:id/addresses
```

> **Nota**  
> También podría pasar que distintos clientes quisieran obtener _distintas representaciones_ de la misma persona, con distintos datos, con sólo la dirección principal, con todas, etc..  
> Vamos a estudiar explícitamente esta cuestión, un poco más adelante.


## REST no es "blanco-o-negro"
Un comentario antes de seguir: creo que no es útil considerar solamente las categorías "RESTful" y "no RESTful" al analizar una API.

Como pasa con muchas cuestiones relacionadas con el desarrollo de software, los principios REST nos dan una guía, que si la seguimos, vamos a obtener ventajas en el producto que estamos concibiendo o desarrollando, en este caso una API.  
Habrá APIs más RESTful que otras. Y también habrá debates y criterios sobre qué opción es "más RESTful" ante un escenario complejo, e incluso sobre hasta qué punto _conviene_ ser "muchísimo muy RESTful" en algunos casos.  

Al respecto, podemos leer sobre el llamado [REST maturity model](https://martinfowler.com/articles/richardsonMaturityModel.html), que habla específicamente de "grados de cumplimiento".

