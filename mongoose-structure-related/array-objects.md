---
layout: default
---

# MongoDB/Mongoose - arrays de objetos
En esta página, vamos a agregar al modelo de sucursales, las fiestas de oficina que hayan tenido lugar en cada sucursal. De cada fiesta se registran dos datos numéricos: la cantidad de asistentes, y el presupuesto. Recordemos (y simplifiquemos, quitando la fecha de cada fiesta) el ejemplo que presentamos en [la presentación de esta sección](./)
```typescript
{
    id: "5fbb0e9495ba0d307cd21374"
    code: "091",
    name: "Cruz del Eje Norte",
    address: "24 de Septiembre 1804",
    area: 400,
    parties: [
        { peopleAttending: 50, budget: 80000 },
        { peopleAttending: 120, budget: 240000 }
    ],
}
```

Mongoose considera cada elemento del array como un documento, y al tipo asociado como un esquema. Mongoose permite dos formas de definir este esquema. 

Una es describir el esquema directamente al definir el atibuto.
```typescript
const AgencySchema = new Schema({
    /* ... otros atributos ... */
    parties: [{ peopleAttending: { type: Number }, budget: { type: Number } }]
});
```

La otra es definir aparte el esquema, y referenciarlo. Esto es lo que se llama _subdocumentos_ en la doc de Mongoose.
```typescript
const PartySchema = new Schema({
    peopleAttending: { type: Number }, budget: { type: Number }
});

const AgencySchema = new Schema({
    /* ... otros atributos ... */
    parties: [{ type: PartySchema }]
});
```

Hay algunas pequeñas diferencias entre estas dos formas de definir un atributo compuesto, que se pueden consultar en [la página de la doc de Mongoose sobre subdocumentos](https://mongoosejs.com/docs/subdocs.html).


## Operaciones
Para **dar de alta una nueva oficina incluyendo fiestas**, simplemente se le da un valor adecuado al atributo `parties`.
```typescript
const newAgency = new this.agencyModel({
    code: '022',
    name: 'Barrio Agote',
    address: 'Mason 2904',
    area: 220,
    parties: [{ peopleAttending: 270, budget: 50000 }, { peopleAttending: 120, budget: 350000 }]
});
newAgency.save();
```

También se pueden ir agrgando fiestas sobre un documento recién creado.
```typescript
const newAgency = new agencyModel({
    code: '007',
    name: 'Yavi',
    address: 'Av. San Martín 328',
    area: 840
});
newAgency.parties.push({ peopleAttending: 180, budget: 730000 });
newAgency.parties.push({ peopleAttending: 28, budget: 380000 });
newAgency.parties.push({ peopleAttending: 535, budget: 880000 });
newAgency.save();
```

Para **agregar fiestas** a una oficina existente, en Mongoose alcanza con agregar  elementos en la lista sobre un documento traido de la base, y darle `save()` al documento.
```typescript
/* es válida cualquier variante de find */
const theAgency = await agencyModel.findById("5fc18e5206d9442f048b92ee");  
theAgency.parties.push({ peopleAttending: 28, budget: 380000 });
theAgency.save();
```

Para **eliminar fiestas** sobre una oficina existente, se puede aplicar la operación `remove()` sobre la fiesta, que es un documento Mongoose y por lo tanto incluye esa operación. Después del (o los) `remove`, _hay que darle `save()` al documento principal_.
```typescript
const theAgency = await agencyModel.findOne({ code: "007" })
const partyToRemove = theAgency.parties.find(party => party.peopleAttending === 1000)
partyToRemove.remove();
theAgency.save();
```

Para **modificar los datos de una fiesta**, procedemos en forma similar a lo que hicimos para eliminar una fiesta: se obtiene la fiesta, se la modifica, se le da `save` al documento principal.
```typescript
const theAgency = await agencyModel.findOne({ code: "007" })
const partyToUpdate = theAgency.parties.find(party => party.peopleAttending === 545)
partyToUpdate.peopleAttending = 535;
theAgency.save();
```

Una alternativa más "cercana" a la BD es usar la operación `updateOne`, y pasar los mismos parámetros que se le pasarían a MongoDB para realizar la misma modificación.
```typescript
const updateResult = await agencyModel.updateOne(
    { code: "007", "parties.peopleAttending": 545 },
    { $set: { "parties.$.peopleAttending": 535 } }
);
```

El `$` en el atributo asociado al `$set`, representa al elemento del array que verifica la condición de búsqueda, en este caso `peopleAttending = 545`; ver [la doc de MongoDB](https://docs.mongodb.com/manual/reference/operator/update/positional/#up._S_).


## Condiciones de búsqueda sobre elementos del array
En las búsquedas de sucursales usando `find`, se pueden incluir condiciones relacionadas con fiestas. Notar que siempre son búsquedas _de sucursales_, o sea, el resultado va a ser una lista de sucursales, que incluyen a las fiestas.

Al igual que para las [listas de valores simples](./array-simple), se asocia el nombre del atributo a los elementos del array. Para listas de objetos, se puede usar notación de punto para acceder a un atributo. P.ej. el resultado de la siguiente consulta son las oficinas que hayan hecho alguna fiesta (o varias) con exactamente 270 invitados.
```typescript
await agencyModel.find({ "parties.peopleAttending": 270 });
```

También se pueden usar condiciones distintas a valor exacto
```typescript
await agencyModel.find({ "parties.peopleAttending": { $lt: 30 } })
```

Pero, **atención**, si se indican varias condiciones, MongoDB las considera por separado, por lo tanto, podría ser que una fiesta cumpla una de las condiciones, y otra fiesta distinta (de la misma sucursal) cumpla la otra. P.ej. ante esta consulta
```typescript
await agencyModel.find(
    { "parties.peopleAttending": 180, "parties.budget": 320000 }
)
```
la siguiente es una posible respuesta (abreviada)
```json
[
    {
        "code": "091",
        "parties": [
            { "peopleAttending": 50, "budget": 80000, "id": "5fbbc7db1ec8d95d14c9cdce" },
            { "peopleAttending": 180, "budget": 320000, "id": "5fc18e8706d9442f048b92f2" }
        ],
        "id": "5fbb0e9495ba0d307cd21374",
    },
    {
        "code": "007",
        "parties": [
            { "peopleAttending": 180, "budget": 730000, "id": "5fc18e5206d9442f048b92ef" },
            { "peopleAttending": 28, "budget": 380000, "id": "5fc18e5206d9442f048b92f0" },
            { "peopleAttending": 545, "budget": 320000, "id": "5fc18e5206d9442f048b92f1" }
        ],
        "id": "5fc18e5206d9442f048b92ee",
    }
]
```
donde la primer oficina sí tiene una fiesta que cumple las dos condiciones, mientras que en la segunda oficina, hay una fiesta con 180 asistentes, _y otra distinta_ con 320000 pesos de presupuesto.

Para especificar en la consulta que tiene que haber un mismo elemento en el array que cumpla todas las condiciones, podemos usar el operador `$elemMatch`, ver [la página en la doc de MongoDB](https://docs.mongodb.com/manual/reference/operator/query/elemMatch/).
```typescript
await agencyModel.find(
    { parties: { $elemMatch: { peopleAttending: 180, budget: 320000 } } }
)
```
La respuesta va a incluir sólo a la primera sucursal
```json
[
    {
        "code": "091",
        "parties": [
            { "peopleAttending": 50, "budget": 80000, "id": "5fbbc7db1ec8d95d14c9cdce" },
            { "peopleAttending": 180, "budget": 320000, "id": "5fc18e8706d9442f048b92f2" }
        ],
        "id": "5fbb0e9495ba0d307cd21374",
    }
]
```
Notar que la sucursal aparece _completa_ en la respuesta, las condiciones se refieren a qué condición debe cumplir _una sucursal_, en este caso, que incluya una fiesta para 180 personas con un presupuesto de 320000 pesos. Veremos más adelante cómo poder seleccionar sólo algunas fiestas de una sucursal.


### Condiciones sobre _todos_ los elementos del array
Veamos ahora un ejemplo distinto: queremos que el resultado de la consulta sean las sucursales en las en que _todas_ las fiestas, hayan ido menos de 40 personas.

Lamentablemente, este humilde redactor no encontró (ni siquiera googleando) cómo expresar esta condición de manera directa en un `find`. La única opción que llevó a una solución es "dar vuelta" la condición: queremos las sucursales en las en que _ninguna_ fiesta haya habido 40 personas o más. Esta condición "dada vuelta" sí se puede expresar, usando el operador `$not`.
```typescript
await agencyModel.find(
    { "parties.peopleAttending": { $not: { $gte: 40 } } }
)
```
El resultado (resumido) que se obtiene es este.
```json
[
    {
        "code": "009",
        "parties": [],
        "id": "5fbb12b977a68b5458a75c82"
    },
    {
        "code": "077",
        "parties": [
            { "peopleAttending": 10, "budget": 8500, "id": "5fc18f7c06d9442f048b92f4" },
            { "peopleAttending": 25, "budget": 12500, "id": "5fc18f8106d9442f048b92f5" }
        ],
        "id": "5fc18f5206d9442f048b92f3"
    }
]
```
La idea de cómo armar este query la saqué de [este post en Stack Overflow](https://stackoverflow.com/questions/23595023/check-if-every-element-in-array-matches-condition).

Volviendo al resultado, vemos que la sucursal `009` "se coló" en el resultado: si no tiene ninguna fiesta, es verdad que ninguna tiene 40 o más participantes.  
Si queremos eliminar las sucursales sin fiestas, tenemos que agregar una condición adicional. Para entender cuál, hay que considerar que el atributo `parties: []` en la primera sucursal es cortesía de Mongoose, en la BD el documento _no tiene_ el atributo `parties`. Veamos el resultado de la misma consulta en la CLI de MongoDB.
```
> db.agencies.find(
    {"parties.peopleAttending": { $not: { $gte: 40 } }}, 
    {_id: 0, code: 1, name: 1, parties: 1}
)

/* resultado */
{ "code" : "009", "name" : "Caballito Sur" }
{ "code" : "077", "name" : "Villa Revol", 
  "parties" : [ 
    { "_id" : ObjectId("5fc18f7c06d9442f048b92f4"), "peopleAttending" : 10, "budget" : 8500 }, 
    { "_id" : ObjectId("5fc18f8106d9442f048b92f5"), "peopleAttending" : 25, "budget" : 12500 } 
] }
```
Obsservamos que la sucursal `009` _no tiene_ un atributo `parties`.
De paso, usamos la [proyección de atributos](../mongoose-performance/proyeccion) para quedarnos sólo con los datos que nos interesan de cada sucursal.

Por eso la condición que vamos a agregar es: el documento tiene el atributo `parties`, y además (por las dudas), su longitud no es 0. La condición completa queda así.

```typescript
await agencyModel.find({ 
    parties: { $exists: 1, $not: { $size: 0} }, 
    "parties.peopleAttending": { $not: { $gte: 40 } } 
})
```
El operador `$exists` permite filtrar los documentos que incluyan (o que no incluyan si se le pasa valor 0) un atributo. El resultado incluye únicamente la sucursal que tiene fiestas.
```json
[
    {
        "code": "077",
        "parties": [
            { "peopleAttending": 10, "budget": 8500, "id": "5fc18f7c06d9442f048b92f4" },
            { "peopleAttending": 25, "budget": 12500, "id": "5fc18f8106d9442f048b92f5" }
        ],
        "id": "5fc18f5206d9442f048b92f3"
    }
]
```


## Acumuladores
Ahora queremos saber, para cada sucursal, el _importe total_ que se gastó en fiestas.  
En una BD relacional en la que la info de las fiestas está en una tabla aparte, una consulta de este estilo se resuelve típicamente usando un `GROUP BY` y operadores de agregación en la cláusula `SELECT`.  
En MongoDB, cualquier consulta que requiera, o bien agregación, o bien joins (que veremos más adelante) se debe resolver (al menos hasta donde conozco) usando la operación `aggregate`, que es distinta del `find`. Mongoose incorpora la operación `aggregate` en sus modelos, llevan el mismo parámetro del `aggregate` de MongoDB.

La operación `aggregate` describe una consulta que procede en _etapas_, en cada etapa se realiza una operación a la que se le pasan parámetros, cada etapa recibe el resultado de la anterior. 
Describimos algunas de las etapas que define MongoDB:
- `$match`: realiza un filtro por condiciones, recibe los mismos parámetros que el `find`.
- `$project`: permite elegir atributos de cada documento, y también agregar acumuladores.
- `$addFields`: agrega atributos a cada documento.
- `$skip` y `$count`: para paginación.

Subrayamos lo de **algunas**: en la documentación a noviembre de 2020 aparecen 31 etapas distintas.

Para nuestro propósito, vamos a usar `$project`, y en uno de sus atributos el operador `$sum`.
```typescript
await agencyModel.aggregate([
    { $project: {
        code: 1, name: 1, _id: 0,
        totalPartyBudget: { $sum: "$parties.budget" }
    } }
]);
```
El resultado tiene esta forma
```json
[
    { "code": "091", "name": "Cruz del Eje Norte", "totalPartyBudget": 400000 },
    { "code": "009", "name": "Caballito Sur", "totalPartyBudget": 0 },
    { "code": "007", "name": "Yavi", "totalPartyBudget": 1430000 }
]
```
Vemos que en el `$project`, se pueden incluir atributos del documento, y también agregar atributos (en este caso `totalPartyBudget`) indicando cómo calcularlos.

Obviamente, estos objetos ya no responden al modelo de sucursales, por eso el resultado del `aggregate` es de tipo `any[]`.

Una aclaración sobre la definición `totalPartyBudget: { $sum: "$parties.budget" }`: el nombre de un atributo (en este caso `parties.budget`) aparece _en el valor_ del atributo `$sum`. Para indicar un atributo del documento en un valor dentro del objeto de la búsqueda, hay que ponerle un `$` adelante, si no lo toma como el string fijo, no como el nombre de un atributo en el documento.

A su vez, algunos operandos incluyen variables, para referirnos a estas variables se usa un "doble pesos", o sea `$$`. Un ejemplo: obtener el presupuesto total usando el operador `$reduce`.
```typescript
await agencyModel.aggregate([
    { $project: {
        code: 1, name: 1, _id: 0,
        totalPartyBudget: { 
            $reduce: { 
                input: "$parties", initialValue: 0, 
                in: { $add: ["$$value","$$this.budget"] }
            }
        }
    } }
]);
```
En este caso, `$$value` es el valor actual del `$reduce`, y `$$this` es el elemento del array.


## Filtrando elementos y/o atributos en el array
Para cerrar, veamos algunos casos en los que queremos obtener sólo parte de la información incluida en el array de fiestas. Estos se consideran casos de [proyección](../mongoose-performance/proyeccion): no estamos filtrando qué documentos, sino qué parte de la info en cada documento.

Si queremos obtener sólo _algunos atributos_, se puede hacer con el parámetro de proyección del `find`.
```typescript
await agencyModel.find({}, { code: 1, name: 1, "parties.peopleAttending": 1, _id: 0 });
```
Este es el resultado
```json
[
    { "code": "091", "name": "Cruz del Eje Norte",
      "parties": [ { "peopleAttending": 50 }, { "peopleAttending": 180 } ]
    },
    { "code": "009", "name": "Caballito Sur" },
    { "code": "007", "name": "Yavi",
      "parties": [ { "peopleAttending": 180 }, { "peopleAttending": 28 }, { "peopleAttending": 545 } ]
    }
]
```

Si queremos obtener sólo _algunos elementos_, tenemos que recurrir al `aggregate`. También nos va a servir la etapa `$project`,  que incluye un operando `$filter`.
```typescript
await agencyModel.aggregate([
    { $project: {
        code: 1, name: 1, _id: 0,
        parties: {
            $filter: {
                input: "$parties",
                as: "party",
                cond: { $lt: ["$$party.peopleAttending", 30] }
            }
        }
    } }
]);
```
El filter indica que de las fiestas, incluya sólo aquellas a las que hayan asistido menos de 30 personas.  
Este `$filter` es análogo a la siguiente expresión en JS/TS.
```typescript
parties.filter(party => party.peopleAttending < 30)
```
El atributo `as: "party"` indica qué nombre va a tener el elemento dentro de la condición, el análogo al parámetro de la arrow function. Como es una variable, hay que accederla como `"$$party"`.

El resultado va a tener esta forma
```json
[
    { "code": "091", "name": "Cruz del Eje Norte", "parties": [] },
    { "code": "009", "name": "Caballito Sur", "parties": null },
    { "code": "007", "name": "Yavi", 
      "parties": [
            {
                "_id": "5fc18e5206d9442f048b92f0",
                "peopleAttending": 28,
                "budget": 380000
            }
      ]
    }
]
```
Observar que **no se están filtrando documentos**: en esta consulta van a aparecer _todas_ las sucursales, lo que estamos especificando es _qué datos de cada sucursal_ queremos obtener. Si **además** queremos obtener información de algunas sucursales, esto lo tenemos que especificar con una etapa separada, que puede ser un `$match`.

Vemos la diferencia entre la sucursal `091`, que tiene fiestas pero ninguna que cumpla la condición, y la `009` que no tiene el atributo `parties`. Para transformar los `null` en `[]` podemos usar el operador `$isNull` ... en una etapa separada, que tome el resultado de este `$project`.
```typescript
await agencyModel.aggregate([
    { $project: {
        code: 1, name: 1, _id: 0,
        parties: {
            $filter: {
                input: "$parties",
                as: "party",
                cond: { $lt: ["$$party.peopleAttending", 30] }
            }
        }
    } },
    { $project: { code: 1, name: 1, parties: { $ifNull: ["$parties", []] } } }
]);
```


## Para practicar
Armar una consulta para obtener las sucursales en las que se haya hecho una fiesta para más de 70 personas, con un presupuesto de menos de 50000 pesos.

Armar una consulta para obtener las sucursales en las que se haya hecho una fiesta para 120 personas, y además una fiesta para 70. Para eso va a venir bien el operador `$all`, ver [la doc de MongoDB](https://docs.mongodb.com/manual/reference/operator/query/all/).

Armar una consulta para obtener las sucursales en las que en cada fiesta, o bien hubo más de 100 personas, o bien se gastaron más de 100000 pesos.  
**Hints**: 
pensar cómo queda la condición "dada vuelta" para usar `$not`, y combinar `$not` con `$elemMatch`.

En la consulta de fiestas donde haya habido menos de 30 personas, incluir solamente las sucursales donde haya habido al menos una fiesta con estas características. Para esto, se puede agregar una etapa `$match` _antes_  de la etapa `$project`.

En la consulta de fiestas donde haya habido menos de 30 personas, de las fiestas que se incluyen, proyectar solamente la cantidad de personas. A mí se me ocurre encadenar dos etapas `$project`.
