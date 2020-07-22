# Ejercicio integrador - un endpoint Post
Para cerrar con esta etapa de la ronda de capacitación, les dejamos un pequeño ejercicio que va a servir (espero) para repasar varios de los temas mencionados.

Se trata de agregar al módulo de solicitudes de cuenta, un endpoint POST que permita agregar una solicitud.

¿Qué hay que hacer? Dos cosas.

**Uno**  
el request handler, o sea un método agregado en el controller, marcado con el decorator `@Post` sin parámetros, para que el endpoint sea
```
POST \account-requests
```
Este método va a tener un parámetro decorado con `@Body` en el que llega un objeto con los datos de la nueva solicitud.  
Definimos un método similar cuando trabajamos con los [pipes de NestJS](../nesjs-basics/pipes.md).  
En este caso, el response es un objeto que tiene un único atributo `id`, cuyo valor es el id que MongoDB le asigna a la nueva solicitud.

**Dos**  
Un método agregado en el servicio, que es invocado por el request handler, y agrega la solicitud en la base. Devuelve el id que MongoDB le asigna a la nueva solicitud.


## Tipos
Nos conviene pensar en los tipos de acuerdo a lo que se analizó acerca de los [tres entornos que estamos manejando](./tipos.md): API, servicio, base.

La propuesta es mantener, en el POST, las mismas definiciones acerca de los tipos de la fecha y el status que se adoptaron para el GET en cada entorno. Por lo tanto, los tipos deberían ser _casi_ iguales a los definidos para el GET. En concreto
- el tipo del `@Body` del request debería ser _casi_ `AccountRequestDto`.
- el tipo del parámetro del método del servicio, _casi_ `AccountRequest`.

El _casi_ es porque no debemos incluir el `id`, que todavía no existe.  
La salida más rápida es definir el `id` como opcional, y usar para el POST los mismos tipos definidos para el GET.  
La más correcta, tener dos interfaces separadas, `AccountRequest` y `AccountRequestProposal`, y lo mismo para los DTO. Usando herencia de interfaces, se puede evitar la repetición de definiciones.

En el método del servicio vamos a tener que crear un objeto que le pasamos al modelo de Mongoose, para que cree el documento al que le vamos a dar `save`. En otras palabras, el método de servicio termina así.
``` typescript
const newMongooseRequest = new this.accountRequestModel(dataForMongoose)
await newMongooseRequest.save()
return newMongooseRequest._id
```

A `dataForMongoose` se le puede dar el tipo `AccountRequestMongooseData`, es para esto que lo definimos por separado de `AccountRequestMongoose`.


## Transformaciones
Se puede armar el encabezado de los métodos en controller y servicio antes de ponerles código, para que queden bien definidos los tipos.

Una vez hecho esto, pensar qué _transformaciones_ de datos hacen falta para poder pasar los datos de controller a servicio, y de servicio al objeto que llamamos `dataForMongoose`. No va a haber ninguna que no se haya hecho cuando [agregamos tipos en el GET](./tipos.md).


## Validaciones
Ahora hay **dos** lugares en los que se pueden definir validaciones. 
Uno es el esquema de Mongoose. 
El otro, la posibilidad de activar p.ej. el `ValidationPipe` sobre el body del request. 

Dónde hacer cada validación, si en un lado, en el otro, o en los dos, es en gran medida una decisión que debe tomarse en cada caso.

Se puede hacer lo siguiente, se recomienda hacerlo en orden.
- comprobar que las validaciones de Mongoose están operativas, ver qué status code se obtiene.
- manejar el error, de acuerdo a lo estudiado sobre [manejo de errores en NestJS](../nestjs-basics/manejo-de-errores.md), para que el status code sea un `400 - Bad Request`, con un mensaje informativo.
- habilitar el `ValidationPipe`, para lo cual habrá que transformar algunas interfaces en clases, por lo que se indicó al [presentar los pipes en NestJS](../nesjs-basics/pipes.md).
- agregar una validación adicional: no se pueden requerir más de 25 aprobaciones. Hacerlo en los dos lugares, primero esquema de Mongoose, probar, después sobre el body.
