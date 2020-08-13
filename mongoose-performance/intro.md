---
layout: default
---

# Performance usando Mongo/Mongoose - resumen de técnicas
Ahora que ya vimos varios detalles sobre el uso de Mongo/Mongoose, hablemos un poco de alternativas para que las operaciones sobre la BD se ejecuten en forma más eficiente.

Hay varias técnicas, ver p.ej. [este post](https://itnext.io/performance-tips-for-mongodb-mongoose-190732a5d382) en donde se describen varias.

Algunas están relacionadas con aspectos de Mongo/Mongoose que ya tratamos, otras las vamos a comentar en las siguientes páginas.


## Selectividad en las búsquedas
Al traer información de una base, muchas veces conviene incluir condiciones en el query para que sea la base la que filtre y al servicio lleguen sólo los datos que se necesitan, en lugar de enviar un volumen mayor de información, lo que implica más tráfico de red más el el trabajo y código adicional de realizar el filtro por código.

Los motores de BD son programas dedicados a operaciones con datos, entre las cuales se destaca la selección o filtro de información. Por eso podemos suponer que, en principio, van a ser tanto o más eficientes que lo que podamos hacer por código. Además, hay que considerar el factor del mayor tráfico por estar enviando datos que van a ser descartados por el servicio.

En el caso extremo, es mucho más eficiente hacer un `findById` 
``` typescript
const miDocumento = elModelo.findById(id)
``` 
en lugar de obtener _todos_ los documentos y filtrar en el servicio.
``` typescript
const miDocumento = elModelo.find({}).find({doc => doc._id === id})
``` 

Tengamos en cuenta también, que la definición de _índices_ permite aumentar la eficiencia de búsquedas donde la selección se hace en la BD, sin necesidad de afectar el código de los servicios.
Respecto de la concurrencia, se puede configurar MongoDB para que corra en varias instancias, ya sea con la información replicada, y/o dividiendo la información en distintos "compartimientos" (los llamados _shards_) que son tratados por instancias separadas.

> **Nota**  
> Esto no es _siempre_ así, si las condiciones son complejas, puede ser conveniente realizar análisis en los servicios para seleccionar y/o consolidar la información que se necesita.
> Pero en casos simples, casi siempre conviene delegar la búsqueda en la base.


### Resumen de criterios de búsqueda utilizados
Lo que sigue es un resumen de las distintas formas de establecer criterios o filtros.

La forma más directa es usar una operación cuyo resultado es un solo documento, de estos usamos `findOne` y `findById`.

Cuando queremos obtener varios documentos, pero no todos, podemos utilizar la condición del `find`, que es un objeto. Vimos varios casos:
- valor directo: `{ requiredApprovals: 3 }`
- comparaciones, p.ej. `{ requiredApprovals: { $gt: 3 } }`, o `{ requiredApprovals: { $in: [3, 5, 8] } }`.
- expresión regular: `{ $regex: ".*bolena.*", $options: 'i'}` (la opción es para que sea case insensitive).
- "or" de condiciones: `{ $or: [{ requiredApprovals: {$in: [3, 5]} }, { status: Status.REJECTED }] }`.


## Usar actualizaciones masivas
Si se sabe que deben modificarse, agregarse o eliminars varios documentos en una misma operación, suele ser más eficiente usar métodos de actualización masiva como `insertMany`, `updateMany` o `deleteMany`, que obtener todos los documentos y eliminarlos "de a uno".


## Y más
Hasta acá lo que ya practicamos. Dejamos a continuación "los títulos" de algunas técnicas que veremos a continuación.

### Lean
Al hacer una operación, típicamente un `find`, se puede indicar una opción `lean`. El efecto es que Mongoose le "pasa" la operación directamente a Mongo, sin intervenir, lo que provoca que la operación sea más rápida.  
Pero claro, hay una desventaja: no aplican las validaciones y agregados del esquema Mongoose, p.ej. los `virtual` y `method`.

### Indices
La definición de _índices_ sobre una BD provoca que algunas búsquedas se hagan **mucho** más rápido.

### No traer todo el documento
Al hacer un `find`, se puede indicar que sólo se obtengan algunos datos de cada documento.

