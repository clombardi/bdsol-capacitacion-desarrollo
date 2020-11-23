---
layout: default
---

# MongoDB/Mongoose - más allá de los documentos planos

En todos los ejemplos que usamos para trabajar con MongoDB/Mongoose, los documentos que almacenamos en las colecciones MongoDB son _objetos planos_. Esto es decir, todos los atributos son valores simples: strings, números, booleanos.

Muchas veces, en nuestro código necesitamos manejar _datos estructurados_, o sea, admitimos que el valor de un atributo pueda ser una lista, un objeto, o combinaciones p.ej. una lista de objetos.


## Datos compuestos en un solo documento
Veamos un ejemplo sencillo: a nuestro modelo que incluye solicitudes de cuenta y sucursales, le queremos agregar los _oficiales de cuenta_, que entre otras cosas evaluarán las solicitudes. De cada oficial nos interesa manejar: su nombre, un nivel que indica el peso de sus evaluaciones, y los nombres de los idiomas que habla. Con un ejemplo: 
```typescript
{
    id: "5fbb132b77a68b5458a75c83",
    name: 'Toshiro Mifune',
    languagesSpoken: ["frances", "japones", "tailandes"],
    level: 1
}
```
Para volcar esta representación de los oficiales en el modelo de BD relacional, hay que crear dos tablas, una para los datos planos del oficial, y otra que relacione cada oficial (representado por su id) con cada uno de los idiomas que habla. La tabla `agents` podría tener esta forma

| **id** | **name** | **level** |
| --- | --- | --- |
| 5fbb132b77a68b5458a75c83 | Toshiro Mifune | 1 |
| 5fba9dad8a65b63af869cdeb | Kwai Chang Caine | 3 |

A esta tabla debe agregarse otra, digamos `agents_languages`, con esta estructura.

| **agentId** | **languageSpoken** |
| --- | --- |
| 5fbb132b77a68b5458a75c83 | frances |
| 5fbb132b77a68b5458a75c83 | japones |
| 5fbb132b77a68b5458a75c83 | italiano |
| 5fba9dad8a65b63af869cdeb | ingles |
| 5fba9dad8a65b63af869cdeb | chino |

En una base de documentos, como MongoDB, el objeto que representa a un oficial puede almacenarse tal como está, en una única colección de `agents`: se simplifica el mapeo y/o el registro de los datos, y se evita la necesidad de un `JOIN` en las consultas.

Un caso un poco más complejo, que también puede registrarse en MongoDB como un documento único, es cuando el valor de un atributo es un _array de objetos_. P.ej. si queremos incluir en el modelo de datos, las fiestas que se hacen en cada sucursal, representando cada sucursal como lo muestra el siguiente ejemplo.
```typescript
{
    id: "5fbb0e9495ba0d307cd21374"
    code: "091",
    name: "Cruz del Eje Norte",
    address: "24 de Septiembre 1804",
    area: 400,
    parties: [
        { date: '2020-08-21', peopleAttending: 50, budget: 80000 },
        { date: '2020-09-04', peopleAttending: 120, budget: 240000 }
    ],
}
```


## Relaciones
La posibilidad de registrar documentos compuestos evita muchas de las relaciones entre distintos documentos que aparecen en el modelo relacional ... pero no todas. 

Siguiendo en el mismo modelo, supongamos que hay que indicar, para cada sucursal, cuáles de las/los oficiales están asociados con la misma, y que cada oficial puede asociarse con varias sucursales (o sea, es una relación `muches-a-muches`).  
La única forma de describir esta relación sin repetir información (o bien la de las sucursales, o bien la de los oficiales), es agregar a los documentos de una colección, referencias a documentos de la otra. P.ej. podemos agregar a cada sucursal una lista con _los ids_ de los oficiales asociados.
```typescript
{
    id: "5fbb0e9495ba0d307cd21374"
    code: "091",
    name: "Cruz del Eje Norte",
    address: "24 de Septiembre 1804",
    area: 400,
    parties: [
        { date: '2020-08-21', peopleAttending: 50, budget: 80000 },
        { date: '2020-09-04', peopleAttending: 120, budget: 240000 }
    ],
    agentIds: [ "5fbb132b77a68b5458a75c83", "5fba9dad8a65b63af869cdeb"]
}
```

Al resolver estas referencias haciendo alguna operación análoga a un `JOIN`, se puede obtener un resultado de esta forma.
```typescript
{
    id: "5fbb0e9495ba0d307cd21374"
    code: "091",
    name: "Cruz del Eje Norte",
    address: "24 de Septiembre 1804",
    area: 400,
    parties: [
        { date: '2020-08-21', peopleAttending: 50, budget: 80000 },
        { date: '2020-09-04', peopleAttending: 120, budget: 240000 }
    ],
    agents: [
        {
            id: "5fbb132b77a68b5458a75c83",
            name: 'Toshiro Mifune',
            languagesSpoken: ["frances", "japones", "tailandes"],
            level: 1
        },
        {
            id: "5fba9dad8a65b63af869cdeb",
            name: "Kwai Chang Caine",
            languagesSpoken: ["ingles", "chino"],
            level: 3
        }        
    ],
}
```

Una variante más compleja es cuando la referencia es uno de los atributos dentro de un array de objetos compuestos. P.ej., si queremos registrar las evaluaciones sobre cada solicitud de cuenta, indicando qué oficial la hizo y cuál fue el resultado (aceptación o rechazo), podemos armar una representación de cada solicitud con esta forma.
```typescript
{
    id: "5fba7d1c5cf6bb23c4ea24b7",
    customer: "Nelly Omar",
    date: "2020-11-17",
    status: "Pending",
    requiredApprovals: 3,
    evaluations: [
        {
            agentId: "5fba98db8a65b63af869cdea",
            decision: "Approve"
        },
        {
            agentId: "5fba9dad8a65b63af869cdeb",
            decision: "Reject"
        }
    ]
}
```


## Esta sección
MongoDB provee varias herramientas para manejar documentos compuestos y relaciones entre documentos. Entre ellas destacamos
- condiciones de consultas sobre documentos anidados y/o arrays dentro de documentos.
- la operación `aggregate` para definir consultas, más potente que el `find`. En particular, permite expresar el `JOIN` de documentos mediante la opción `$lookup`.
- índices sobre valores en arrays dentro de un documento.

A su vez, Mongoose también incluye herramientas en el mismo sentido. 
- Los esquemas permiten definir el tipo de un atributo como un array, como un _subdocumento_, o como una combinación de ambos.
- Permite indicar que un atributo es una _referencia_ a otro documento, en la misma colección o en otra.
- En las consultas permite indicar, mediante la opción `populate`, que incluya en el resultado los documentos referenciados, o sea el análogo del `JOIN` en una consulta relacional.

Cada una de estas herramientas tiene una cantidad _enorme_ de matices, variantes y combinaciones. En esta sección vamos a hacer una _descripción inicial_ de estos temas, mencionando _**algunos** ejemplos_ de características que pueden aprovecharse. Para seguir analizando y aprendiendo, nos remitimos a la [doc de Mongoose](https://mongoosejs.com/docs/) y a [la de MongoDB](https://docs.mongodb.com/manual/).

