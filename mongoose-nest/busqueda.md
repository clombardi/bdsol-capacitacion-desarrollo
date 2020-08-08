---
layout: default
---

# Búsqueda de documentos
En esta página, vamos a aplicar lo que estudiamos sobre [búsqueda de documentos en MongoDB/Mongoose](../mongoose/busqueda-basicos) a nuestro módulo NestJS.

Para esto, vamos a admitir dos query params en el endpoint de búsqueda de documentos, que permita filtrar por status y/o por (nombre de) cliente. Si se envía un valor de nombre de cliente, se busca en cualquier parte del nombre. P.ej. si se especifica `ana`, encontraría ya sea `analía Amandi` (en minúscula), `Mariana Suárez`, o `Lucas Antezana`.

O sea, vamos a admitir p.ej. los siguientes `Get` requests.
```
/account-requests?customer=ana&status=Accepted
/account-requests?customer=ana
/account-requests?status=Accepted
/account-requests
```

Vamos a usar el mismo request handler, o sea el mismo método de controller; y el mismo método de servicio. Veamos qué (pocos) agregados se necesitan.


## Controller: sólo recibir y transmitir los query params
En el request handler sólo hay que agregar la declaración de los `@Query()` params, y enviar el objeto formado por esos parámetros al servicio.

``` typescript
@Get()
async getAccountRequests(
    @Query() conditions: AccountRequestFilterConditions
): Promise<AccountRequestDto[]> {
    const requests = await this.service.getAccountRequests(conditions);
    return requests.map(request => { 
        return { ...request, date: request.date.format(stdDateFormat) } 
    });
}
``` 
con esta interface `AccountRequestFilterConditions`.
``` typescript
export interface AccountRequestFilterConditions {
    customer?: string,
    status?: string
}
``` 

Así como se recibe el objeto con las condiciones, se envía al servicio. En el ejemplo en que se envían ambos valores, el objeto `conditions` tiene esta forma:
``` javascript
{
    customer: "ana", 
    status: "Accepted"
}
``` 


## Servicio: requiere una transformación de las condiciones
Observamos que el formato de este objeto es parecido al del parámetro por el que se especifican condiciones de búsqueda en el `find()` de MongoDB.
- el objeto debe incluir atributos con los mismos nombres
- la búsqueda por `status` es por valor exacto, por lo tanto es correcto enviar directamente el valor por el que se está buscando.
- para la búsqueda por `customer` hay que usar una expresión regular, por lo tanto hay que transformar el valor recibido en un objeto.  
En el ejemplo, reemplazar `{ customer: "ana" }` por `{ customer: { $regex: ".*ana.*" } }`.

Con esta transformación, directamente enviamos el objeto como parámetro del `find`. Y eso es todo lo que hay que hacer.
 ``` typescript
    async getAccountRequests(conditions: AccountRequestFilterConditions): Promise<AccountRequest[]> {
        const findConditions: any = {}
        if (conditions.status) {
            findConditions.status = conditions.status
        }
        if (conditions.customer) {
            findConditions.customer = { $regex: `.*${conditions.customer}.*` }
        }
        const mongooseData = await this.accountRequestModel.find(findConditions);
        return mongooseData.map(mongooseReq => { return {
            id: mongooseReq._id,
            customer: mongooseReq.customer,
            status: mongooseReq.status as Status,
            date: moment.utc(mongooseReq.date),
            requiredApprovals: mongooseReq.requiredApprovals
        }})
    }
``` 

> **Nota**  
> El de `findConditions` es un caso en el la decisión de poner `any` como tipo, puede ser menor el riesgo de errores que el engorro de escribir el tipo correspondiente.


## Moraleja
En muchos casos, resulta sencillo hacer una búsqueda acotada en Mongo, con la consiguiente ventaja de eficiencia de que viaja menos información de la base al servicio, respecto de hacer un `filter` por código en el servicio.


## Ejercicios
Agregar un endpoint GET `/account-requests/:id`, que busque una solicitud por el ID de Mongo. Devolver el status code adecuado si no encuentra ninguna solicitud con el id indicado. Usar `findOne`.

Lograr que la búsqueda de customer sea case-insensitive, o sea que buscando por `ana` también encuentre p.ej. `Analía Amandi` o `Rosa Anand`. En [la página sobre regex en la doc de MongoDB](https://docs.mongodb.com/manual/reference/operator/query/regex/) está la info necesaria.

Agregar validación de los parámetros de búsqueda. El status tiene que ser uno de los valores del `enum Status`. El nombre debe tener al menos 3 caracteres. Usar `ValidationPipe`. 

Agregar la posibilidad de buscar por rango de fechas, con los query params `fromDate` y `toDate`. Acá hay que armar una condición compleja sobre el atributo `date`.
