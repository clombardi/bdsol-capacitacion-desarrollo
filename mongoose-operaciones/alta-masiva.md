---
layout: default
---

# Alta masiva
El siguiente desafío es exponer un endpoint que permita el _alta masiva_ de un conjunto de solicitudes de cuenta. Obviamente, es un `POST`.
El body tiene esta forma.

``` typescript
export interface AccountRequestMassiveAdditionDTO {
    date: string,
    defaultRequiredApprovals?: number,
    requestDetails: {
        customer: string,
        requiredApprovals?: number
    }[]
}
```

Es decir, que
- todas las solictudes se van a registrar con la misma fecha.
- para las que no indiquen cantidad de aprobaciones requeridas, se usa el default. Si tampoco se pone el default, se considera que se requieren 3 aprobaciones.
- (además) todas las solicitudes se crean con estado Pendiente.

Hablemos de los elementos que nos van a servir.


## Operación en MongoDB/Mongoose
Así como tenemos `updateMany` y `deleteMany`, los modelos de Mongoose (y las colecciones de MongoDb) ofrecen la operación `insertMany`. En este caso, el parámetro es una lista de objetos en el formato en que se definió el esquema. Esta es la [doc de Mongoose](https://mongoosejs.com/docs/api/model.html#model_Model.insertMany).

El método `insertMany` tiene varias opciones interesantes. Algunas indican qué hacer si alguno de los documentos a agregar es erróneo, sobre estas sólo comentamos que existen. Hay otra relacionada con la performance, que vamos a mencionar un poco más adelante. 


## Un poco de planficación
Recordemos las interfaces relacionadas con el alta de una solicitud.

``` typescript
export interface AccountRequestProposal {
    customer: string,
    status: Status,
    date: moment.Moment, 
    requiredApprovals: number
}

export interface AccountRequestMongooseData {
    customer: string,
    status: string,
    date: number,
    requiredApprovals: number
}
```

`AccountRequestProposal` corresponde a lo que recibe el método del provider que realiza el alta 
``` typescript
async addAccountRequest(req: AccountRequestProposal): Promise<string> {
    // implementación
}
```

`AccountRequestMongooseData` coincide con el esquema de Mongoose
``` typescript
export const AccountRequestSchema = new mongoose.Schema({
    customer: { type: String, required: true },
    status: { type: String, enum: Object.values(Status) },
    date: Number,
    requiredApprovals: { type: Number, default: 3 }
})
```

Por lo tanto, el nuestro caso, el `insertMany` espera una lista de `AccountRequestMongooseData`. 
Hay que definir quién se va a encargar de transformar un `AccountRequestMassiveAdditionDTO`  en esta lista.

Consideramos que esta transformación tiene que ver con la forma de los datos de entrada, y por lo tanto se la vamos a asignar mayoritariamente al controller.

Por lo tanto
- el controller va a transformar el `AccountRequestMassiveAdditionDTO` en una lista de `AccountRequestProposal`. De esta forma mantenemos la coherencia en la interfaz de los métodos del provider.
- el provider hace el pasito de transformar cada `AccountRequestProposal` en un `AccountRequestMongooseData`, la misma transformación que se hace en el alta individual.


## Transformación en el controller - hay que programar un poco
Para que el método del controller me quedara corto, definí una función separada
``` typescript 
export function transformIntoAccountRequestProposal(
        massiveAdditionData: AccountRequestMassiveAdditionDTO
): AccountRequestProposal[] {
    // implementación
}
```
En rigor, el `export` no hace falta, si definimos la función en el mismo archivo del controller. Pero esta función es un lindo candidato para armar algún test específico, y ahí sí hace falta el `export`.

El método de controller queda cortito.
``` typescript
@Post('/massiveAddition')
async accountRequestMassiveAddition(
        @Body() massiveAdditionData: AccountRequestMassiveAdditionDTO
): Promise<AccountRequestMassiveAdditionResultDTO> {
    const processedAdditionData: AccountRequestProposal[] = 
        transformIntoAccountRequestProposal(massiveAdditionData);
    const addedRequestsCount = await this.service.addManyAccountRequests(
        processedAdditionData
    );
    return { addedRequestsCount }
}
```

El tipo de respuesta incluye solamente la cantidad de solicitudes agregadas.
``` typescript
export interface AccountRequestMassiveAdditionResultDTO {
    addedRequestsCount: number
}
```


## Estamos para implementar
Eso, el desafío es implementar el endpoint, con todo lo que haga falta.