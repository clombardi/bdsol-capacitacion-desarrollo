---
layout: default
---

# MongoDB/Mongoose - referencias
El siguiente paso es agregar los oficiales relacionados con cada sucursal. En rigor lo que vamos a incluir son FK a la colección de oficiales.

Para definir en un esquema Mongoose, que el valor de un atributo es un id, se le pone el tipo `Schema.Types.ObjectId`. A la definición del atributo se le puede agregar un elemento `ref`, que indica a cuál de los modelos mapeados apunta este id. En nuestro ejemplo: 
```typescript
export const AgencySchema = new Schema({
    code: { type: String },
    /* ... otros atributos ... */
    agents: [{ type: Schema.Types.ObjectId, ref: "Agent" }]
});
```

Las operaciones se manejan como se describió al hablar de [arrays de objetos](./array-objects), donde en lugar de incluir / agregar / quitar objetos, se trabaja con referencias.  
Hay dos formas de generar/obtener referencias. Una es obteniendo el objeto relacionado de la BD y pedirle el `_id`.
```typescript
const theAgent = await agentModel.findById(agentId);
theAgency.agents.push(theAgent._id);
```
La otra es generando un id a partir de un string, usando la operación `Types.ObjectId`, donde `Types` se importa del package `mongoose`.
```typescript
theAgency.agents.push(Types.ObjectId(agentId));
```

P.ej. así se crea una sucursal con dos oficiales cargados.
```typescript
const newAgency = new agencyModel({
    code: '078',
    name: 'Barrio Yapeyú',
    address: 'Cabeza de Tigre 228',
    area: 555,
    agents: [Types.ObjectId("5fba9dad8a65b63af869cdeb"), Types.ObjectId("5fbb132b77a68b5458a75c83")]
});
newAgency.save();
```
Recordemos que la BD, al no ser relacional, _no va a verificar que los valores de id que se pasan correspondan a oficiales_. Mongoose tampoco incorpora esta validación.

Para relaciones _muchos-a-uno_ o _uno-a-uno_, el tipo del atributo va a ser un `ObjectId` en lugar de una lista, y en vez de manipular referencias en una lista, se asignan al atributo. P.ej. si cada sucursal se asocia a un único oficial, se puede asignar el oficial de una sucursal así
```typescript
office.agent = agent._id
```
o así
```typescript
office.agent = Types.ObjectId(agentId)
```


## Populate
Mongoose brinda una operación sobre las queries llamada **populate**, que simplifica en varias situaciones (no siempre), la obtención de documentos relacionados, o sea el `JOIN`.
La operación lleva como parámetro el atributo del modelo sobre el que se hace la query, cuyo valor es un id, o una lista de ids, que hacen referencia a otro/s documento/s; en nuestro ejemplo, es el atributo `agents` del `AgencyModel`.  
El efecto del `populate` es reemplazar cada id por el documento relacionado. En nuestro ejemplo, se reemplazará cada id de oficial, por el documento completo de la colección de oficiales.  
Para esta operación, Mongoose necesita que esté seteado el `ref` en el atributo, para saber en qué colección/modelo buscar los documentos relacionados.  

En nuestro ejemplo, para incluir los documentos completos de los oficiales asociados a una sucursal en lugar de únicamente los ids, se indica de esta forma:
```typescript
await agencyModel.find(/* ... condiciones ... */).populate('agents');
```
donde en lugar de `find`, puede ser `findOne` o `findById`.

Veamos la diferencia de una consulta con o sin `populate`.

Para obtener una sucursal sin incluir las fiestas asociadas, podemos definir la siguiente consulta.
```typescript
await agencyModel.findOne({ code: '007'}).select({ parties: 0 })
```
obteniendo un resultado de esta forma
```json
{
    "id": "5fc18e5206d9442f048b92ee",
    "code": "007",
    "name": "Yavi",
    "address": "Av. San Martín 328",
    "area": 840,
    "agents": [
        "5fbbb86549bdcd33548038ef",
        "5fba9dad8a65b63af869cdeb"
    ]
}
```
donde de cada oficial se obtiene el `id`, el dato registrado en la colección de sucursales.

Si a la misma consulta le agregamos la indicación de que "popule" los oficiales
```typescript
await agencyModel.findOne({ code: '007'}).select({ parties: 0 }).populate('agents');
```
el resultado cambia a esto
```json
{
    "id": "5fc18e5206d9442f048b92ee",
    "code": "007",
    "name": "Yavi",
    "address": "Av. San Martín 328",
    "area": 840,
    "agents": [
        { 
            "id": "5fbbb86549bdcd33548038ef", "level": 1, "name": "Vassili Kandinski",  
            "languagesSpoken": [ "frances", "ruso", "italiano" ]
        },
        {
            "id": "5fba9dad8a65b63af869cdeb", "level": 3, "name": "Kwai Chang Caine",
            "languagesSpoken": [ "ingles", "chino" ],
        }
    ]
}
```
En esta versión, de cada oficial se incluye la información completa que se obtiene de la colección de oficiales.

El populate de Mongoose tiene **muchas** variantes, en lo que sigue vamos a ver _algunas_.
El resto se puede consultar en [la página de la doc de Mongoose sobre populate](https://mongoosejs.com/docs/populate.html). 


## Condiciones de búsqueda sobre documentos relacionados
Usando el `populate`, podemos incorporar los documentos relacionados en el _resultado_ de una consulta. En algunos casos, puede ser necesario incluir _condiciones_ relacionadas con estos documentos:
- incluir sólo parte de la información sobre estos documentos, p.ej. mostrar sólo los oficiales que hablan japonés.
- filtrar la búsqueda, en este caso de sucursales, de acuerdo a condiciones sobre los documentos relacionados. P.ej., buscar las sucursales que tengan al menos un oficial relacionado de nivel superior a 1.

La operación `populate` permite expresar algunas de estas condiciones. En otros casos, vamos a tener que recurrir al `agregate`, que incluye una etapa que permite expresar un `JOIN`.

### Filtrar sobre la lista de oficiales de cada sucursal
La operación `populate` acepta que se le pasen condiciones sobre los documentos relacionados que se van a incluir.  
Para especificar estas condiciones, en lugar de pasar como parámetro solamente el nombre del atributo con el id, hay que pasar un objeto, donde se indica como `path` este nombre de atributo, y se puede indicar un `match` con las mismas variantes que los parámetros de búsqueda en un `find`. Estas condiciones se aplican _sobre los documentos relacionados_, en nuestro ejemplo sobre los oficiales.

P.ej. para obtener para cada sucursal, sólo los oficiales que hablan japonés y/o chino, se puede especificar esta búsqueda.
```typescript
await agencyModel.find().select({ parties: 0 }).populate({ 
    path: 'agents', 
    match: { $or: [{ languagesSpoken: "chino" }, { languagesSpoken: "japones" } ] } 
});
```

Está claro que esta consulta no establece ninguna condición sobre las sucursales. Si una sucursal no tiene oficiales que hablen chino o japonés, va a aparecer en el resultado, con un array vacío de oficiales.
P.ej. las sucursales `007` y `009` que tienen estos datos
```json
[
    {
        "id": "5fc18e5206d9442f048b92ee",
        "code": "007",
        "name": "Yavi",
        "address": "Av. San Martín 328",
        "area": 840,
        "agents": [
            {
                "id": "5fbbb86549bdcd33548038ef",
                "name": "Vassili Kandinski",
                "level": 1,
                "languagesSpoken": [ "frances", "ruso", "italiano" ],
            },
            {
                "id": "5fba9dad8a65b63af869cdeb",
                "name": "Kwai Chang Caine",
                "level": 3,
                "languagesSpoken": [ "ingles", "chino" ]
            }
        ]
    },
    {
        "id": "5fbb12b977a68b5458a75c82",
        "code": "009",
        "name": "Caballito Sur",
        "address": "Yavi 328",
        "area": 128,
        "agents": [
            {
                "id": "5fba98db8a65b63af869cdea",
                "name": "Marco Denevi",
                "level": 1,
                "languagesSpoken": [ "ingles", "italiano", "frances" ]
            }
        ]
    }
]
```
en la consulta anterior se ven así.
```json
[
    {
        "id": "5fc18e5206d9442f048b92ee",
        "code": "007",
        "name": "Yavi",
        "address": "Av. San Martín 328",
        "area": 840,
        "agents": [
            {
                "id": "5fba9dad8a65b63af869cdeb",
                "name": "Kwai Chang Caine",
                "level": 3,
                "languagesSpoken": [ "ingles", "chino" ]
            }
        ]
    },
    {
        "id": "5fbb12b977a68b5458a75c82",
        "code": "009",
        "name": "Caballito Sur",
        "address": "Yavi 328",
        "area": 128,
        "agents": []
    }
]
```
La sucursal `007` aparece con el oficial que habla chino, y la `009` también se incluye en el resultado, con una lista vacía de oficiales.

### Filtrar la búsqueda de sucursales de acuerdo a datos de sus oficiales
Vayamos al otro caso: queremos obtener _las sucursales_ que tengan asociado al menos un oficial con nivel mayor a 1. En este caso queremos que las sucursales que no tengan oficiales con las condiciones pedidas, no aparezcan en el resultado.

La operación `populate` no contempla que se puedan utilizar los datos de los documentos relacionados para filtrar la búsqueda de los documentos principales. 
Una forma de resolver este tipo de consultas es recurrir a la operación `aggregate` de MongoDB, que describimos [al hablar de listas de objetos](./array-objects). Entre las etapas (_stages_) que se pueden usar en un `aggregate`, está `$lookup`, que permite definir el `JOIN` de sucursales a oficiales; los documentos relacionados se agregan como un nuevo atributo al documento principal. En una etapa posterior, se puede definir un filtro sobre el nuevo atributo, en una etapa `$match`.

La consulta planteada puede resolverse de esta forma.
```typescript
await agencyModel.aggregate([
    {
        $lookup: {
            from: "agents",
            localField: "agents",
            foreignField: "_id",
            as: "agentData"
        }
    },
    { $match: { "agentData.level": { $gt: 1 } }},
    { $project: { parties: 0, __v: 0 }}
]);
```
En los cuatro parámetros del `$lookup` se indican:
- `from`: con qué colección hacer el `JOIN`
- `localField`: el atributo del documento principal que tiene la FK.
- `foreignField`: el atributo del documento relacionado que tiene el mismo valor.
- `as`: el nombre del atributo que se agrega al resultado de la etapa.

Vemos que la siguiente etapa es un `$match`, que puede definir criterios de búsqueda sobre el atributo `agentData`, agregado en la etapa anterior.

Mostramos un posible resultado de este `aggregate`
```json
[
    {
        "_id": "5fc18e5206d9442f048b92ee",
        "code": "007",
        "name": "Yavi",
        "address": "Av. San Martín 328",
        "area": 840,
        "agents": [
            "5fbbb86549bdcd33548038ef",
            "5fba9dad8a65b63af869cdeb"
        ],
        "agentData": [
            {
                "_id": "5fba9dad8a65b63af869cdeb",
                "name": "Kwai Chang Caine",
                "level": 3,
                "languagesSpoken": [ "ingles", "chino" ]
            }
            {
                "_id": "5fbbb86549bdcd33548038ef",
                "name": "Vassili Kandinski",
                "level": 1,
                "languagesSpoken": [ "frances", "ruso", "italiano" ]
            }
        ]
    },
    {
        "_id": "5fc26759914d2e18b0a55c43",
        "code": "078",
        "name": "Barrio Yapeyú",
        "address": "Cabeza de Tigre 228",
        "area": 555,
        "agents": [
            "5fba9dad8a65b63af869cdeb",
            "5fbb132b77a68b5458a75c83"
        ],
        "agentData": [
            {
                "_id": "5fba9dad8a65b63af869cdeb",
                "name": "Kwai Chang Caine",
                "level": 3,
                "languagesSpoken": [ "ingles", "chino" ]
            }
            {
                "_id": "5fbb132b77a68b5458a75c83",
                "name": "Toshiro Mifune",
                "level": 1,
                "languagesSpoken": [ "frances", "japones", "tailandes" ]
            }
        ]
    }
]
```
Vemos que se agrega el atributo `agentData`, y que sólo aparecen las sucursales asociadas a un oficial de nivel superior a 1. Por otro lado, de las sucursales que se incluyen en la respuesta aparecen todos los oficiales: la etapa `$match` filtra documentos, pero no qué datos de cada documento incluir. Para esto usamos la etapa `$project`, en la que se eliminan algunos datos.


## Para practicar
Armar una consulta en la que se incluyan todas las sucursales, y para cada una, sólo los oficiales con nivel > 1.

Agregar al endpoint `GET /agencies` dos query params, uno que indica si incorporar o no los datos de las fiestas, otro que indique si incorporar o no la info de los oficiales asociados.

Agregar una relación entre las oficinas y su gerente, que es un oficial.

En la consulta resuelta con un `aggregate`, agregar a la etapa `$project` lo necesario para que sólo incluya, para cada sucursal, los oficiales de nivel superior a 1. Para esto se puede usar el operador `$filter` que describimos en [la página sobre arrays de objetos](./array-objects).

Armar una consulta en la que se obtengan sólo las sucursales que tengan oficiales asociados, y en las que _todos_ los oficiales tengan nivel > 1.

Armar una consulta en la que se obtengan las sucursales con más de 300 metros cuadrados de superficie (el atributo `area`), y de estas, los oficiales que hablen exactamente 3 idiomas.
