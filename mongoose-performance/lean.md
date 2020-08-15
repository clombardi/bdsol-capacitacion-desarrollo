---
layout: default
---

# Lean - Mongoose se corre a un costado
Como lo indicamos al [presentar las distintas técnicas para performance](./intro), la opción de `lean` permite ganar eficiencia en las operaciones que se hacen mediante Mongoose, en particular los `find`. Pero no es gratis, se pierden todos los agregados que se hagan al esquema.

La sintaxis para usar esta opción es muy sencilla: a la operación de búsqueda
``` typescript
await <model>.find(<opciones>);
```
sencillamente se le agrega `.lean()` _detrás. O sea:
``` typescript
await <model>.find(<opciones>).lean();
```

Y eso es todo. Veámoslo con un ejemplo, siempre sobre el ejemplo de las solicitudes de cuenta.


## Recordemos el contexto
Transcribimos el esquema y la interfaz que expone el servicio.
``` typescript
export const AccountRequestSchema = new mongoose.Schema({
    customer: { type: String, required: true },
    status: { type: String, enum: Object.values(Status) },
    date: Number,
    requiredApprovals: { type: Number, default: 3, max: 1000 }
})

AccountRequestSchema.virtual('isDecided').get(
    function(): boolean { return [Status.ACCEPTED, Status.REJECTED].includes(this.status) }
);

AccountRequestSchema.method({
    hasDate: function(): boolean { return !!(this.date && this.date != 0) },
    month: function(): number | undefined {
        return this.hasDate() ? moment(this.date).utc().month() + 1 : undefined 
    },
})

export interface AccountRequest {
    customer: string,
    status: Status,
    date: moment.Moment, 
    requiredApprovals: number
    id: string,
    month: number, 
    isDecided: boolean
}
```

Este es el método en el servicio.
``` typescript
async getAccountRequests(): Promise<AccountRequest[]> {
    const mongooseData = await this.accountRequestModel.find({});
    return mongooseData.map(mongooseReq => {
        return {
            id: mongooseReq._id,
            customer: mongooseReq.customer,
            status: mongooseReq.status as Status,
            date: moment.utc(mongooseReq.date),
            requiredApprovals: mongooseReq.requiredApprovals,
            month: mongooseReq.month(),
            isDecided: mongooseReq.isDecided
        }
    })
}
```
Observamos que el servicio aprovecha `month()` e `isDecided`, definidos como un `method` y un `virtual` respectivamente en el esquema Mongoose.


## Aplicando _lean_ - primer intento
Aplicamos la mínima modificación necesaria para ganar eficiencia, en el método del servicio.
``` typescript
async getAccountRequests(): Promise<AccountRequest[]> {
    const mongooseData = await this.accountRequestModel.find({}).lean();
    return mongooseData.map(mongooseReq => {
        return {
            id: mongooseReq._id,
            customer: mongooseReq.customer,
            status: mongooseReq.status as Status,
            date: moment.utc(mongooseReq.date),
            requiredApprovals: mongooseReq.requiredApprovals,
            month: mongooseReq.month(),
            isDecided: mongooseReq.isDecided
        }
    })
}
```
Compila perfectamente, no da problema de tipos. Pero el test falla (¡qué bueno que teníamos tests!).

![al usar mal lean, falla el test](./images/wrong-lean-makes-test-failure.jpg)

El error se produce porque al usar `lean`, no se aplican los `virtual` y `method` agregados al esquema Mongoose.  

El efecto del `lean` es que Mongoose _prácticamente no interviene_, se limita a pasarle la consulta a MongoDB, devolviéndole al servicio la información que obtiene de la base sin procesarla, modificarla ni hacerle agregados. 
En particular, no se activan ni los `method` ni los `virtual`. Por eso, los objetos que devuelve el `find` no incluyen ni la función `month()` ni el atributo `isDecided`, sólo se incluyen el `_id` y los atributos que forman parte del documento Mongo, en nuestro caso, `customer`, `status`, `date` y `requiredApprovals`; en el formato en el que están en la base.

Por eso, cuando el servicio le pide el `month()` al resultado del `find`, se genera el error: al hacer la búsqueda en "modo `lean`", los documentos se obtienen en una versión no "retocada" por Mongoose, y por lo tanto,  `mongooseReq.month()` es `undefined`.


## Qué hacer - pasar responsabilidades de Mongoose a los providers
Para poder "mantener" la ventaja de eficiencia que nos da el `lean`, tenemos que prescindir de `virtual` y `method`. Hay dos acciones que podemos tomar.
1. pensar si _realmente_ aporta incluir los datos calculados en la respuesta, o lo estábamos haciendo "de chiche".
2. si decidimos mantener los datos, hacer que los calcule _el provider_ en lugar del servicio.

En concreto: a la definición del esquema le quitamos los `virtual` y `method`, y agregamos el código que calcula los atributos a agregar en el provider.

``` typescript
async getAccountRequests(): Promise<AccountRequest[]> {
    const mongooseData = await this.accountRequestModel.find({}).lean();
    return mongooseData.map(mongooseReq => {
        const theDate = moment.utc(mongooseReq.date);
        const theStatus = mongooseReq.status as Status
        return {
            id: mongooseReq._id,
            customer: mongooseReq.customer,
            status: theStatus,
            date: theDate,
            requiredApprovals: mongooseReq.requiredApprovals,
            month: theDate.month() + 1,
            isDecided: [Status.ACCEPTED, Status.REJECTED].includes(theStatus)
        }
    });
}
```

aquí vemos que el método del provider incluye el cálculo de `month` e `isDecided`. Si hay que hacer esta transformación en varios métodos del provider, se puede "sacar" a una función.
``` typescript
function mongooseToModel(mongooseReq: AccountRequestMongoose): AccountRequest {
    const theDate = moment.utc(mongooseReq.date);
    const theStatus = mongooseReq.status as Status
    return {
        id: mongooseReq._id,
        customer: mongooseReq.customer,
        status: theStatus,
        date: theDate,
        requiredApprovals: mongooseReq.requiredApprovals,
        month: theDate.month() + 1,
        isDecided: [Status.ACCEPTED, Status.REJECTED].includes(theStatus)
    }
}

// ... en el provider ...
async getAccountRequests(): Promise<AccountRequest[]> {
    const mongooseData = await this.accountRequestModel.find({}).lean();
    return mongooseData.map(mongooseToModel)
}
```


## Conclusiones - ¿qué se hace, se usan los agregados de Mongoose o no?
Tal vez (esto es para pensarlo y para debatirlo con su arquitecte amigue) en una aplicación en la que el acceso a los datos se concentra en providers bien definidos, el atractivo de las características que agregan los esquemas de Mongoose sea menor que con una arquitectura distinta. 
Esto sí, hay que mantener la prolijidad de siempre pedir los datos al provider que corresponde.

**Un detalle importante**  
Si se están obteniendo documentos que después van a modificarse usando `save`, en estos casos **no se puede** usar `lean`, porque el resultado es un objeto "plano", no un documento de Mongoose; y por lo tanto no incluye la operación `save()`.

Un aspecto que conviene tener en cuenta es la _carga_ estimada de cada colección. Si en una colección se espera una cantidad en el orden de los cientos de documentos, seguramente la degradación de performance por agregar características en los esquemas es moderada. Para colecciones con millones de documentos, puede ser intolerable.


## Otro escenario para usar _lean_: alta masiva
La operación `insertMany` tiene una opción `lean`, que tiene el efecto análogo de que Mongoose no actúa. En este caso, _no se ejecutan las validaciones definidas en el esquema_, lo que quiere decir que si se incluyen documentos que no pasan las validaciones, se van a insertar. 

En este caso, el `lean` va dentro de un objeto `options`, que es un segundo parámetro opcional del `insertMany`.
``` typescript
const mongooseResult = await this.accountRequestModel.insertMany(
    mongooseRequestsData, { lean: true }
);
``` 

Respecto del ejemplo del [alta masiva](../mongoose-operaciones/alta-masiva), si se incluye una solicitud para la que se piden más de 1000 aprobaciones, si se hace el `insertMany` con la opción `lean` prendida, se va a registrar, justamente porque **no se ejecutan las validaciones definidas en el esquema**. Úsese a su riesgo. Otra vez, hay que balancear carga contra prestaciones.

> **Nota de implementación**  
> Si se escribe el `insertMany` con la opción _lean_ exactamente como está indicado, _no va a compilar_, al menos con las versiones actuales al 14/08/2020, mientras estoy escribiendo esto.  
> El problema es que `@types/mongoose`, que es lo que TS usa para el chequeo de tipos, está actualizada (en su versión actual, la 5.7.36) hasta la versión 5.7.13 de Mongoose.
> ``` typescript
> // Type definitions for Mongoose 5.7.13
> // Project: http://mongoosejs.com/
> ```
> mientras que la opción `lean` de `insertMany` [fue agregada a Mongoose en la versión 5.8.8](https://github.com/Automattic/mongoose/pull/8507).
> 
> En mi entorno, tengo la versión 5.9.9 de Mongoose. O sea, tengo la opción `lean`, pero _el chequeo de tipos_ no la reconoce.
>
> Cómo lo arreglé: by-passeando el chequeo de tipos
> ``` typescript
> const mongooseResult = await (this.accountRequestModel as any).insertMany(
>     mongooseRequestsData, { lean: true }
> );
> ``` 
>
> Dejo el comentario porque me pareció un caso interesante, que podría llegar a pasar con alguna otra librería.


