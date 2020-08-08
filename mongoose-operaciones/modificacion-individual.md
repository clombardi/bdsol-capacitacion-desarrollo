---
layout: default
---

# Un endpoint para acceder a una modificación individual
Volvemos al trabajo especifico con Mongo y Mongoose. En particular, vamos a ampliar lo que vimos sobre _operaciones que modifican la base_ (altas / bajas / modificaciones).

Lo que hagamos, será siempre _en el contexto de un servicio NestJS_.  
Esto nos va a permitir que, al tiempo que experimentamos algunas otras características de MongoDB y Mngoose, podamos practicar otros temas que tratamos, en particular definición de API, tests, y manejo de errores.


## Qué hicimos, cuál es el siguiente paso
Lo que cubrimos de Mongoose incluyó algo de [operaciones](../mongoose/mongoose-cuatro-conceptos): vimos cómo hacer un alta y cómo modificar información de un documento que trajimos de la base. 
Luego, al integrar Mongoose en un servicio NestJS, planteamos como [ejercicio](../mongoose-nest/ejercicio-post) armar un endpoint que agrega una solicitud de cuenta.

En esta página, vamos a completar este pequeño panorama agregando un endpoint que apunta a _modificar_ una solicitud de cuenta. Específicamente, a cambiar el status de una solicitud determinada, de `Analysing` a `Pending`.
Vamos paso a paso, abordando los distintos elementos que forman parte de la resolución de un nuevo requerimiento.


## El nuevo endpoint
Tenemos que definir método/verbo y URL.

Empecemos por el método: estamos _modificando_ un documento. Los candidatos son `PUT` y `PATCH`.  
La modificación ¿es idempotente? A primera vista parecería que sí: el efecto es siempre poner el valor `Pending` en el atributo `status`. Pero si lo hago dos veces sobre la misma solicitud, la segunda debería dar error, porque ya no está `Analysing`. 
Por otro lado, no modifica todo el recurso, toca solamente un atributo. 
Por todo esto, nos inclinamos por `PATCH`.

Ahora la URL.  
La operación se refiere a un recurso específico. Por lo tanto, la URL debe incluir un identificador de recurso. Utilicemos el id que genera MongoDB.
Con esto le dijimos cuál es el recurso a modificar, falta indicar en qué consiste la modificación.
Como es fijo, lo ponemos en la URL. Nos queda así
```
PATCH /account-requests/:id/set-as-pending
```

Finalmente, la respuesta. Como es un solo recurso, podemos devolver la representación standard del mismo.


## El provider
Ese se los dejamos a ustedes.


## El servicio
Definimos un método que toma el id, realiza la modificación, y devuelve un `AccountRequest`, o sea, la representación de una solicitud para el servicio. Ver lo que debatimos sobre [tipos en un servicio NestJs](../mongoose-nest/tipos).

Lo primero que tenemos que hacer es obtenerlo de la base, buscando por el id. Recordemos que para Mongoose, el atributo se llama `_id`. Una vez que lo encontramos, le cambiamos el dato que queremos cambiar y le damos `save`.
``` typescript
const accountRequestDb = await this.accountRequestModel.findOne({ _id: id });
accountRequestDb.status = Status.PENDING;
await accountRequestDb.save()
```

Sobre esta base, nos faltan dos cosas
1. qué pasa si el id es inválido, o sea, el `findOne` devuelve `null`. Hagamos que salga con un status `404 - Not Found`.
1. armar el valor de retorno. Hay que transformar el documento Mongoose en un objeto que cumpla la interface definida para el servicio.

El método completo nos queda así.
``` typescript
async setAsPending(id: string): Promise<AccountRequest> {
    const accountRequestDb = await this.accountRequestModel.findOne({ _id: id });
    if (!accountRequestDb) {
        throw new NotFoundException(`Account request ${id} not found`);
    }
    accountRequestDb.status = Status.PENDING;
    await accountRequestDb.save()
    return {
        id: accountRequestDb._id,
        customer: accountRequestDb.customer,
        status: accountRequestDb.status as Status,
        date: moment.utc(accountRequestDb.date),
        requiredApprovals: accountRequestDb.requiredApprovals,
        month: accountRequestDb.month(),
        isDecided: accountRequestDb.isDecided
    }
}
```

## Hacerlo andar - "make it work"
Agregar el método de controller, que debe devolver el tipo DTO definido.
Probar con Postman.


## Un poquito de "make it right"
Una vez que nos sacamos la ansiedad de que funciona, podemos hacerle mejoras. 
- Completamos la implementación, agregando un mínimo test.
- Funcionalidad: la definición es "cambiar el status de `Analysing` a `Pending`". ¿Y si la solicitud está en un estado distinto a `Analysing`? Salir con un status code `403 - Forbidden` en este caso.
- Un refactor: el código que transforma un documento Mongoose en un objeto `AccountRequest` está en el método que devuelve una lista de solicitudes, y ahora también en este. Definir una función que hace la transformación para una solicitud, y llamarla en los dos métodos del provider.
- Otro refactor: ver si no quedó algo similar en el controller, respecto de la transformación de `AccountRequest` a DTO.


## Otro ejercicio
Agregar un nuevo atributo a las solicitudes de cuenta: el saldo esperado (`expectedBalance`) de la cuenta que se solicita abrir.
Les propongo hacerlo en este orden.
- Primero agregar el atributo a las interfaces y esquema Mongoose, ver todos los lugares donde da error. Hacemos un alto para agradecer al tipado de TS.
- Después ir corrigiendo cada error. Yo lo haría primero en el service, después en el controller, después lo que quede.
- Para el final se pueden dejar los tests. También van a aparecer errores, arreglarlos y ejecutarlos. Si nos quedó algo rojo, corregimos.

Una vez hecho esto, agregar un endpoint que permita cambiar el saldo esperado de una solicitud, con estas consideraciones: no se puede hacer para solicitudes rechazadas, el importe que se indica no puede ser negativo.

