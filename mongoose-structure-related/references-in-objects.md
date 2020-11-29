---
layout: default
---

# MongoDB/Mongoose - referencias incluidas en objetos
En esta página, vamos a agregar las evaluaciones de solicitudes de cuenta. Para cada evaluación se registra: qué oficial la hizo, y cuál es la recomendación, si aceptar o rechazar la solicitud.

Se decide registrar las evaluaciones como un atributo en cada solicitud, que se describe como un _subdocumento_, esto es, se define un esquema separado, y se usa ese esquema para definir el tipo del atributo `evaluations` en el esquema de solicitudes.  

El código queda así.
```typescript
export enum EvaluationDecision {
    APROVE = 'Approve',
    REJECT = 'Reject'
}

export const AccountApplicationEvaluationSchema = new Schema({
    agentId: { type: Schema.Types.ObjectId, ref: 'agents' },
    decision: { type: String, enum: Object.values(EvaluationDecision) }
});

export const AccountApplicationSchema = new Schema({
    customer: { type: String, required: true },
    status: { type: String, enum: Object.values(Status), default: Status.PENDING },
    date: Number,
    requiredApprovals: { type: Number, max: 1000 },
    evaluations: { type: [AccountApplicationEvaluationSchema] }
});
```

Para las operaciones de alta, baja y modificación, se utilizan los mecanismos descriptos [al tratar sobre listas de objetos](./array-objects), dando valor al atributo `agentId` de cada evaluación según las opciones indicadas en [la página sobre referencias](./references).

P.ej. este método de provider agrega una evaluación sobre una solicitud existente
```typescript
async addEvaluation(
    applicationId: string, newEvaluationData: AddAccountApplicationEvaluationDTO
): Promise<AccountApplicationDTO> {
    const { agentId, decision } = newEvaluationData;
    const application = await this.accountApplicationModel.findById(applicationId);
    const newEvaluation: AccountApplicationEvaluation = { 
        agentId: Types.ObjectId(agentId), decision 
    }
    application.evaluations.push(newEvaluation);
    application.save();
    return application.toObject();
}
```

Para más detalles, se puede consultar la [página sobre subdocuments en la doc de Mongoose](https://mongoosejs.com/docs/subdocs.html).


## Consultas que involucran objetos relacionados
Para incorporar al resultado de una consulta la información sobre el oficial que realizó cada evaluación, pueden aplicarse las dos técnicas que describimos en [la página sobre relaciones](./references): la operación `populate` de Mongoose, y la etapa `$lookup` del `aggregate` de MongoDB.

Un ejemplo: este `find` obtiene todas las solicitudes, reemplazando el valor del atributo `agent` en cada evaluación, por el documento completo del evaluador.
```typescript
await accountApplicationModel.find().populate("evaluations.agent"); 
```
Este es el resultado, acotado a una solicitud.
```json
{
    "id": "5fba7d1c5cf6bb23c4ea24b7",
    "status": "Pending",
    "customer": "Nelly Omar",
    "requiredApprovals": 3,
    "isDecided": false,
    "date": "2020-11-17",
    "evaluations": [
        {
            "decision": "Approve",
            "agent": {
                "id": "5fba98db8a65b63af869cdea",
                "name": "Marco Denevi",
                "level": 1,
                "languagesSpoken": [ "ingles", "italiano", "frances" ]
            },
            "id": "5fbae7cd79b1cd1e80caa050"
        },
        {
            "decision": "Approve",
            "agent": {
                "id": "5fba9dad8a65b63af869cdeb",
                "name": "Kwai Chang Caine",
                "level": 3,
                "languagesSpoken": [ "ingles", "chino" ],
            },
            "id": "5fbaf68d9f514f483896d420"
        }
    ]
}
```

Para agregar la información sobre los oficiales que evaluaron cada solicitud, un primer impulso podría ser armar un `aggregate` con una etapa `$lookup`.
```typescript
await accountApplicationModel.aggregate([
    {
        $lookup: {
            from: "agents",
            localField: "evaluations.agent",
            foreignField: "_id",
            as: "agentData"
        }
    }
]);
```
**Peeeero** en este caso, en lugar de modificarse el valor de `agents`, se agrega un nuevo atributo `agentData` en cada solicitud, con la lista de los oficiales de cada evaluación.
Veamos un posible resultado, acotado a una solicitud.
```json
{
    "id": "5fba7d1c5cf6bb23c4ea24b7",
    "status": "Pending",
    "customer": "Nelly Omar",
    "requiredApprovals": 3,
    "date": "2020-11-17",
    "evaluations": [
        {
            "id": "5fbae7cd79b1cd1e80caa050",
            "decision": "Approve",
            "agent": "5fba98db8a65b63af869cdea"
        },
        {
            "id": "5fbaf68d9f514f483896d420",
            "decision": "Approve",
            "agent": "5fba9dad8a65b63af869cdeb"
        }
    ],
    "agentData": [
        {
            "id": "5fba98db8a65b63af869cdea",
            "name": "Marco Denevi",
            "level": 1,
            "languagesSpoken": [ "ingles", "italiano", "frances" ]
        },
        {
            "id": "5fba9dad8a65b63af869cdeb",
            "name": "Kwai Chang Caine",
            "level": 3,
            "languagesSpoken": [ "ingles", "chino" ],
        }
    ]
}
```
Para asociar el oficial con cada evaluación, hay que buscar en `agentData` el oficial que coincida por `id`, con el `id` de la evaluación. Esta asociación viene resuelta en el resultado del `populate`.

En lugar de hacer esta asociación por código, se puede armar un `aggregate` que nos brinde la información con el mismo formato que el `populate`, en objetos planos en lugar de documentos Mongoose (y sin los virtual). Pero ... el `aggregate` queda _bastante_ complejo.
```typescript
await accountApplicationModel.aggregate([
    { $unwind: { path: "$evaluations", preserveNullAndEmptyArrays: true } },
    {
        $lookup: {
            from: "agents",
            localField: "evaluations.agent",
            foreignField: "_id",
            as: "agentData"
        }
    },
    { $set: { "evaluations.agent": { $arrayElemAt: ["$agentData", 0] } } },
    { $unset: "agentData" },
    {
        $group: {
            _id: "$_id",
            data: { $first: "$$ROOT" },
            evaluations: { $push: "$evaluations" }
        }
    },
    {
        $replaceRoot: {
            newRoot: { $mergeObjects: [
                "$data", { evaluations: { $cond: [
                    { $eq: [{ $arrayElemAt: ["$evaluations", 0] }, {}] }, [], "$evaluations"
                ] } }
            ] }
        }
    }
]);
```
Qué hace esta operación:
- el `$unwind` desnormaliza: se genera un resultado para cada evaluación
- el `$lookup` aplicado a los resultados desnormalizados, asocia cada evaluación con el oficial que la realizó
- el `$set` siguiente es porque el atributo que agrega el `$lookup` siempre es un array.
- el `$unset` elimina el atributo `agentData` que ya incorporamos dentro de la evaluación.
- el `$group` vuelve a normalizar, agrupando los resultados por id de solicitud. Por limitaciones del `$group`, quedan los datos de la evaluación en `data`, y las evaluaciones en `evaluations`.
- en el `$replaceRoot` final, se ensambla el resultado. El condicional es para prever el caso de las solicitudes que no tienen evaluaciones.


### Entonces ¿qué hacemos, populate o aggregate?
Está claro que el `populate` es _mucho_ mas sencillo que el `aggregate` que hubo que armar.  
Pero no todo es felicidad del lado del `populate`:
- no permite establecer condiciones de búsqueda sobre las solicitudes, basadas en datos en las evaluaciones, como ya comentamos en [la página sobre referencias](./references).
- tal como lo indica [la doc de Mongoose](https://mongoosejs.com/docs/populate.html), las referencias se resuelven en queries separadas. Esperemos que haga una sola query para toda la consulta, y no una por solicitud, o peor, una por evaluador. No encontré documentación al respecto. 

Por otro lado, un `aggregate` complejo podría ser costoso para el motor de BD. De acuerdo a los niveles de distribución de la BD (MongoDB contempla los conceptos de _réplica_ y _shard_), puede resultar conveniente armar un `aggregate` más sencillo, y hacer un post-procesamiento en el server de negocio.


## Para practicar
Armar una consulta cuyo resultado sean las evaluaciones hechas por un determinado oficial a partir de su id.

Armar una consulta cuyo resultado sean las solicitudes que hayan sido evaluadas por al menos un oficial de nivel > 1.

Armar una consulta cuyo resultado sea una lista de objetos `{customer, requiredApprovals, acceptedCount, rejectedCount}`  para cada solicitud.

Armar una consulta que incluya, para cada solicitud, solamente sus evaluaciones de rechazo.


**Desafío Mongo**  
Supongamos que se quiere cambiar el nombre del atributo `decision` de las evaluaciones, poniéndole `advice`. Hacer las modificaciones en el código puede requerir un poco de paciencia, pero no es difícil. 
El desafío es escribir una o varias sentencias Mongo, que hagan la modificación.  
A mí me salió con dos sentencias, primero agregué el atributo `advice` con el valor que tuviera `decision`, y después eliminé el atributo `decision`. Para el primero usé una expresión `$map` en una etapa `$set` en un `aggregate`, el segundo es un `$unset` con una sintaxis especial para indicar "en todos los elementos del array `evaluation`.  
Ahora que lo escribo, tal vez alcance con un `$map` más complejo ... aunque horrible.
