---
layout: default
---

# MongoDB/Mongoose - arrays de documentos simples
En esta sección vamos a agregar al modelo de oficiales, los idiomas que habla cada oficial, que se describen como listas de strings, p.ej. `["frances", "japones", "tailandes"]`.

Para indicarle a Mongoose que el valor de un atributo es un array de valores simples (strings, números, booleanos), alcanza con encerrar el tipo entre corchetes en la definición de atributo. P.ej. `{ type: [String] }` para arrary de string, lo mismo para números o booleanos. El esquema de `Agents` queda así.
```typescript
const AgentSchema = new Schema({
    name: { type: String, required: true, minlength: 4, maxlength: 120 },
    languagesSpoken: { type: [String] },
    level: { type: Number, min: 1, max: 4, default: 1 }
});
```

Para agregar un oficial, entre los datos que se le pasan al nuevo documento, el valor de `languagesSpoken` se carga como un array de JS/TS.
```typescript
const newAgentData = {
    name: 'Bruce Lee',
    languagesSpoken: ["ingles", "japones", "chino"],
    level: 2
}
const newAgent: Agent = new agentModel(newAgentData);
await newAgent.save();
```

Los documentos que se obtienen desde la BD mediante `find`, tienen como valor del atributo un array. P.ej. al evaluar
```typescript
const foundAgent = await agentModel.findById("5fbc1c3b15d00224cc27a2c1");
console.log('Objeto obtenido desde la BD -----------------')
console.log(foundAgent.toObject());
console.log('Cantidad de lenguajes que habla -----------------')
console.log(foundAgent.languagesSpoken.length);
```
se obtiene lo siguiente
```typescript
Objeto obtenido desde la BD -----------------
{
  languagesSpoken: [ 'ingles', 'japones', 'chino' ],
  level: 2,
  _id: 5fbc1c3b15d00224cc27a2c1,
  name: 'Bruce Lee',
  __v: 0
}
Cantidad de lenguajes que habla -----------------
3
```

## Consultas sobre los valores del array
MongoDB incluye varios operadores de búsqueda que apuntan a valores que son arrays. En la doc, hay una explicación general en [esta página del manual](https://docs.mongodb.com/manual/tutorial/query-arrays/) y el detalle de los operadores en [esta página en la referencia](https://docs.mongodb.com/manual/reference/operator/query-array/).

Veamos un par de ejemplos.

Lo más fácil es filtrar por un elemento, o sea, obtener los documentos para los que el array incluye un elemento determinado; en el ejemplo, los oficiales que hablan un idioma determinado. Para eso, se asocia directamente al nombre del atributo, en este caso `languageSpoken`, el valor por el que se quiere buscar, o sea el idioma.  
Tal vez es más fácil con un ejemplo: el resultado de 
```typescript
await agentModel.find({ languagesSpoken: 'italiano' })
```
es la lista de oficiales que hablan italiano.  
Notar que **la misma** sintaxis que para un atributo de valor simple indica una búsqueda por valor exacto, si el atribuo es un array, indica búsqueda por elemento.

Si el valor asociado al atributo es un array, ahí si se considera una búsqueda por valor exacto ... _que incluye el orden de los elementos_. O sea, el resultado de
```typescript
await agentModel.find({ languagesSpoken: ['ingles', 'italiano', 'frances'] })
```
es la lista de oficiales para los que la lista de lenguajes que hablan sea _exactamente_ la indicada, o sea, tienen que estar esos idiomas _en ese orden_. Si un oficial tiene como lista de lenguajes `['italiano', 'ingles', 'frances']`, no va a aparecer en el resultado.

### Un par más de casos

Para obtener los oficiales que hablen varios idiomas, p.ej. inglés e italiano, se puede hacer con el operador `$and`, pero también el operador `$all`.
```typescript
await agentModel.find({ languagesSpoken: { $all: ['italiano', 'ingles'] } })
```
**OJO**  
el operador `$all` tiene algunas sutilezas que se pueden consultar en [la página correspondiente en la referencia](https://docs.mongodb.com/manual/reference/operator/query/all/).

Para obtener los oficiales que hablen exactamente dos idiomas, se puede utilizar el operador `$size`.
```typescript
await agentModel.find({ languagesSpoken: { $size: 2 } })
```
y **OJO** otra vez
Este operador sirve solamente por consultas por _tamaño exacto_. Lamentablemente, para obtener p.ej. los oficiales que hablan 3 ó más idiomas, no se puede hacer con una consulta de esta forma.
```typescript
await agentModel.find({ languagesSpoken: { $size: { $gte: 2 } } })
```
Para estas consultas, hay que recurrir a la operación `aggregate` que veremos bastante más adelante. Para algunos casos puntuales, se puede hacer algún truco, ver p.ej. en los ejercicios.


## Ejercicios
Armar una consulta cuyo resultado sean los oficiales que hablan, o bien inglés, o bien francés, al menos uno de los dos.

Armar una consulta cuyo resultado sean los oficiales que hablan italiano y exactamente dos idiomas más.

Armar una consulta cuyo resultado sean los oficiales que hablan, o bien 3, o bien 4 idiomas. 

Sabiendo que cada oficial habla al menos un idioma, armar una consulta cuyo resultado sean los oficiales que hablan dos o más (ayuda: usar el `$not`).

