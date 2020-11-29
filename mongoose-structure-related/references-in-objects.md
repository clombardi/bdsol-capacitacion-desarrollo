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
    return dbToDTO(application);  
    /* dbToDTO es una función que genera el DTO de respuesta a partir del documento Mongoose, 
       aplicando toObject() y haciendo algunos ajustes */
}
```

Para más detalles, se puede consultar la [página sobre subdocuments en la doc de Mongoose](https://mongoosejs.com/docs/subdocs.html).

