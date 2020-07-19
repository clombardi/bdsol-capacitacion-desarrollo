# Búsqueda de documentos
En la página anterior, vimos que el resultado de la operación `<modelo>.find()`, _sin parámetros_, es un array que incluye _todos_ los documentos de la colección correspondiente.

A la misma operación se le pueden pasar _criterios de búsqueda_, para restringir los documentos que se obtienen. Estos criterios se pasan como parámetro en el `find`, o sea:
``` javascript
<modelo>.find(<criterio>) 
```
Los mismos criterios se pueden aplicar a la operación `findOne()`, y también a otras operaciones similares como p.ej. `findOneAndDelete()`. Ver el listado de [operaciones sobre un modelo](https://mongoosejs.com/docs/api/model.html).

**Importante**  
Mongoose _no_ define un lenguaje propio para describir los criterios de búsqueda. Se usa la especificación _de MongoDB_, respecto de esto Mongoose se limita a "pasarle" los criterios a Mongo sin transformarlos.  
Esto puede ser distinto en librerías que apunten a ser compatibles con distintas bases de datos.


## Nuestra primera búsqueda
Los criterios de búsqueda en Mongo _son un mundo_, hay muchísimas opciones, pensadas para poder efectura distintas operaciones en forma eficiente sobre bases con muchos datos.  
Encontramos la documentación en [el sitio de MongoDB](https://docs.mongodb.com/manual/reference/operator/query/).

> **Comentario**  
> Un aspecto a considerar cuando se piensa en la _eficiencia_ en una operación que involucra datos que están en una base, es si conviene que el procesamiento lo haga la base, o bien si conviene enviar datos más "crudos" al servidor de lógica de negocio, y que el procesamiento se haga ahí.  
> Un ejemplo extremo: para obtener la suma de los puntos obtenidos por un millón de jugadores en un juego online, suele ser más eficiente hacer la suma en la base y que al servicio viaje un número, que pasar el millón de objetos al servicio para que resuelva el cálculo. La razón es el tiempo de transmisión de los datos de la base al servicio.   
> Sobre esto hay mucho escrito, pensado y estudiado; aquí nos limitamos a mencionar este tema para agregarlo a la visión de "cosas que se pueden pensar" al resolver un servicio.

En esta página, nos limitamos a mostrar algunos casos sencillos.

Recordemos la definición de esquema y modelo.
``` javascript
const accountRequestSchema = new mongoose.Schema({
    customer: { type: String, required: true },
    status: { type: String, enum: ['Pending', 'Analysing', 'Accepted', 'Rejected'] },
    date: Number,
    requiredApprovals: Number
})
const accountRequestModel = mongoose.model('AccountRequest', accountRequestSchema);
```

Empecemos por algo bien básico: queremos las solicitudes que estén aprobadas, o sea que el valor del atributo `status` sea `Accepted`. 
``` javascript
await accountRequestModel.find({ status: 'Accepted' })
``` 
Los criterios se definen como un objeto, en el que se especifican las condiciones para cada atributo. Sólo se incluyen los atributos sobre los que se quiere definir condiciones: en este caso, la única condición es sobre el `status`.

### Valor exacto
La condición más simple es "valor exacto", para eso alcanza con poner el valor.
Obviamente, este valor puede no ser una constante.
``` javascript
async function findByStatus(status) {
    return await accountRequestModel.find({ status })
}
``` 

### Un solo documento
El resultado del `find` siempre es una lista, aunque haya un solo documento que cumpla la condición. Veamos un ejemplo de consulta y su respuesta.
``` javascript
await accountRequestModel.find({ customer: 'Pedro Navaja' })

[
    {
        _id: 5e929cfbfc8c3d6370dc0c1c,
        customer: 'Pedro Navaja',
        status: 'Pending',
        requiredApprovals: 21,
        date: 1585699200000,
        __v: 0
    }
]
``` 

Para obtener el resultado, hay que agregar `[0]`.
``` javascript
(await accountRequestModel.find({ customer: 'Pedro Navaja' }))[0]

{
    _id: 5e929cfbfc8c3d6370dc0c1c,
    customer: 'Pedro Navaja',
    status: 'Pending',
    requiredApprovals: 21,
    date: 1585699200000,
    __v: 0
}
``` 

Si sabemos que va a haber exactamente un resultado, o si puede haber muchos pero queremos uno cualquiera, podemos usar `findOne` y nos "ahorramos el `[0]`".
``` javascript
await accountRequestModel.findOne({ customer: 'Pedro Navaja' })

{
    _id: 5e929cfbfc8c3d6370dc0c1c,
    customer: 'Pedro Navaja',
    status: 'Pending',
    requiredApprovals: 21,
    date: 1585699200000,
    __v: 0
}
``` 

## Comparaciones
Para cualquier condición que no sea "valor exacto", se define _un objeto_ con las condiciones, y ese objeto se asocia al atributo.  

``` javascript
await <model>.find({ <atributo>: {<condiciones>} })
``` 

Cada condición que se quiera especificar, va a estar asociada a un atributo _dentro del objeto de condiciones_. Los nombres de estos atributos suelen empezar con el signo `$`.

Vamos con un ejemplo: las comparaciones de mayor, menor, etc. sobre atributos numéricos, se especifican en [esta página](https://docs.mongodb.com/manual/reference/operator/query-comparison/). Para "mayor", el atributo es `$gt`.

Tal vez con el ejemplo se ve más fácil: para obtener las solicitudes que requieren de más de dos aprobaciones, la expresión es
``` javascript
await accountRequestModel.find({ requiredApprovals: { $gt: 2 } })
``` 
Lo que decíamos: en el objeto que se pasa como parámetro al `find`, el valor correspondiente al atributo `requiredApprovals` es _otro objeto_, que especifica las condiciones correspondientes.

La condición de "valor distinto a", se puede manejar usando el atributo `$ne`. Por ejemplo, podemos obtener las solicitudes que **no** están aprobadas de esta forma.
``` javascript
await accountRequestModel.find({ status: { $ne: 'Accepted' } })
``` 

## Expresiones regulares
Para atributos String, podemos usar [expresiones regulares](https://devopedia.org/images/article/173/6028.1557317770.jpg) que permiten hacer muchas búsquedas intersantes. El atributo para especificar una expresión regular es `$regex`, que está descripto [acá](https://docs.mongodb.com/manual/reference/operator/query/regex/index.html).

Por ejemplo, podemos obtener las solicitudes hechas por clientes de apellido "Bolena" así.
``` javascript
await accountRequestModel.find({ customer: { $regex: 'Bolena$' }})
``` 
(o sea, que el valor _termine_ con `Bolena`).



## Para ejercitar
Pregunta: si no hay _ningún_ documento que cumpla las condiciones en un `find`, ¿cuál será la respuesta? ¿Y para el `findOne`?

Buscar las solicitudes que requieran de entre 3 y 5 aprobaciones.  
**Hint**: para poner varias condiciones que aplican al mismo atributo y deben cumplirse todas (o sea, hay que considerar el "AND" entre las condiciones) se pueden separar con comas dentro del objeto asociado al atributo. O sea:  
`{<atributo>: {<cond1>: <valor1>, <cond2>: <valor2>, ...}}`

Buscar las solicitudes que tengan el mismo status que la solicitud de Ana Bolena.
Vale hacer dos queries.

Buscar las solicitudes de clientes cuyo nombre empiece con A.

Buscar las solicitudes de clientes cuyo apellido (o sea, el final del valor de `customer`) sea "Bolena" o "Navaja".
