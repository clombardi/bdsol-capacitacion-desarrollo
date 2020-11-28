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
Intro.

### Filtrar sobre la lista de oficiales de cada sucursal
Esto se puede hacer con el populate.

### Filtrar la búsqueda de sucursales de acuerdo a datos de sus oficiales
Acá tenemos que ir a un aggregate `$lookup`.


## Para practicar
Armar distintas versiones de consultas relacionadas con los niveles de los oficiales asociados a cada sucursal
1. obtener todas las sucursales, y para cada una, sólo los oficiales con nivel > 1.
1. obtener sólo las sucursales que tengan al menos un oficial con nivel > 1, de cada sucursal todos los datos.
1. obtener sólo las sucursales que tengan al menos un oficial con nivel > 1, de cada sucursal, incluir sólo esos oficiales.
1. obtener sólo las sucursales que tengan oficiales asociados, y en las que _todos_ los oficiales tengan nivel > 1.

Agregar al endpoint `GET /agencies` dos query params, uno que indica si incorporar o no los datos de las fiestas, otro que indique si incorporar o no la info de los oficiales asociados.

Agregar una relación entre las oficinas y su gerente, que es un oficial.