# Algo más sobre esquemas
Como se indicó [al presentar los esquemas](./mongoose-cuatro-conceptos.md), además de la configuración de cada atributo, se pueden incluir definiciones adicionales. 
El efecto de varias de estas definiciones es funcionalidad _a los documentos_ que se crean a partir de (los modelos creados a partir de) un esquema.

En esta página analizamos dos tipos de definiciones, como ejemplo de qué se puede obtener configurando adecuadamente los esquemas Mongoose.  
Más en general, esperamos que este material sirva como disparador para el estudio de capacidades que brinda Mongoose más allá del manejo básico que describimos en las páginas anteriores.


## Virtuals - atributos calculados
A la definición de un esquema se le pueden definir los llamados "virtuals" [en la documentación](https://mongoosejs.com/docs/guide.html#virtuals).   
Un virtual agrega _un atributo_ a cada documento obtenido a partir del esquema. La definición del virtual incluye una función que calcula el valor para el atributo que se agrega. 

Vamos con un ejemplo: queremos agregar un atributo que indica si una solicitud tiene una decisión tomada (ya sea de aceptarla o rechazarla), o no. Alcanza con agregar lo siguiente.
``` javascript
accountRequestSchema.virtual('isDecided').get(
    function() { return ['Accepted', 'Rejected'].includes(this.status) }
);
```
En la función que se pasa en el `get`, `this` se refiere al documento al que se le está agregando un atributo.

> **Atención**  
> La función que se pasa en el `get` _no puede ser una arrow function_, tiene que definirse usando `function`. Supongo que la razón está relacionada con quién es `this`, reconozco que no lo estudié.

Con esta definición, los documentos que se obtengan mediante `find` o sus variantes, van a incluir el atributo `isDecided`.
``` javascript
const request = await accountRequestModel.findOne({ customer: 'Pedro Navaja' });
request.status              // 'Pending'
request.isDecided           // false
```

Incluso los documentos que se creen a partir de un model, ya van a tener este atributo.
``` javascript
const newRequest = new accountRequestModel({
    customer: 'Juana Inés de Asbaje',
    status: 'Accepted'
});
request.isDecided           // true
```

### En el toObject
Los virtuals **no** aparecen por defecto en el resultado de `toObject` ...
``` javascript
const request = await accountRequestModel.findOne({ customer: 'Pedro Navaja' });
request.toObject()

{
    requiredApprovals: 21,
    _id: 5e929cfbfc8c3d6370dc0c1c,
    customer: 'Pedro Navaja',
    status: 'Pending',
    date: 1585699200000,
    __v: 0
}
```
... pero se puede pedir que los incluya, mediante una opción que se le pasa al `toObject()` por parámetro.
``` javascript
const request = await accountRequestModel.findOne({ customer: 'Pedro Navaja' });
request.toObject({virtuals: true})

{
    requiredApprovals: 21,
    _id: 5e929cfbfc8c3d6370dc0c1c,
    customer: 'Pedro Navaja',
    status: 'Pending',
    date: 1585699200000,
    __v: 0,
    isDecided: false,
    id: '5e929cfbfc8c3d6370dc0c1c'
}
```
Ahora sí aparece `isDecided` ... y también `id`, que parece ser un virtual que Mongoose agrega por defecto.


## Methods - comportamiento agregado
Además de atributos, a los documentos se le pueden agregar métodos, que se especifican en el esquema. 

Vamos a usar esta capacidad para habilitar transformaciones relacionadas con la fecha de una solicitud. En el esquema, esta fecha se definió como un número, la idea es que esté expresada en el formato [Unix time](https://en.wikipedia.org/wiki/Unix_time). Vamos a agregar métodos para poder obtener, o asignar, la fecha tanto como un String en formato `YYYY-MM-DD`,  como usando un objeto `moment` provisto por [la librería MomentJS](https://momentjs.com/).

Para esto, agregamos lo siguiente a la definición del esquema.
``` javascript
accountRequestSchema.method({ 
    setDateFromString: function(str) { 
        const theMoment = moment.utc(str, 'YYYY-MM-DD')
        this.date = theMoment.valueOf()
    },
    setDateFromMoment: function(someMoment) {
        this.date = someMoment.valueOf()
    },
    hasDate: function () { return !!(this.date && this.date != 0) },
    dateAsMoment: function() { 
        return this.hasDate() ? moment(this.date).utc() : undefined 
    },
    dateAsString: function() { 
        return this.hasDate() ? moment(this.date).utc().format('YYYY-MM-DD') : undefined 
    }
})
```

Esta definición agrega los cinco métodos definidos, a los documentos que se creen u obtengan.
``` javascript
const doc = new accountRequestModel({
    customer: 'Juana Inés de Asbaje',
    status: 'Analysing',
    requiredApprovals: 7
});

const date = moment({ year: 2020, month: 4, day: 21})
doc.hasDate()                   // false

doc.setDateFromMoment(date);    
doc.hasDate()                   // true
doc.date                        // 1590030000000
doc.dateAsString()              // "2020-05-21"
```

Como estos son métodos y no atributos, no hay forma de que el `toObject` los incorpore automáticamente. Tendremos que agregarlos "a mano".
``` javascript
{...doc.toObject(), dateAsString: doc.dateAsString()}

{
    requiredApprovals: 3,
    _id: 5f14a07fff27304fe86964b5,
    customer: 'Juana Inés de Asbaje',
    status: 'Analysing',
    date: 1590030000000,
    dateAsString: '2020-05-21'
}
```


## Para cerrar
En esta página vimos que en un esquema se pueden definir más cosas, además de configurar atributos.

Otra observación que puede ser útil es que se nota la diferencia entre un documento de Mongoose, y el objeto que sólo incluye los atributos que definimos. 
O sea, la diferencia entre
``` javascript
{
    customer: 'Juana Inés de Asbaje',
    status: 'Accepted'
}
```
y
``` javascript
new accountRequestModel({
    customer: 'Juana Inés de Asbaje',
    status: 'Accepted'
})
```
la "capa" que agrega Mongoose, le inyecta funcionalidad agregada al objeto que representa al documento. Inclusive, simplifica su modificación como vimos [al describir las operaciones](./mongoose-cuatro-conceptos.md).  
Configurando el esquema, le podemos agregar potencia a los documentos Mongoose.


## Para practicar
Todo esto es sobre el `accountRequestSchema`.

En realidad, tanto `dateAsMoment` como `dateAsString` pueden definirse como virtuals, definiendo los métodos para obtener _y para cambiar_ el valor, de acuerdo a lo que se explica en [la documentación sobre esquemas](https://mongoosejs.com/docs/guide.html#virtuals). Hacerlo.

Agregar un método `reject()` que cambie el status a `Rejected`.

Agregar un atributo `receivedApprovals` que arranque en 0. Agregar un método `approve()` que, si el status es `Analysing`, entonces le sume uno a este atributo, y si el valor luego de esto coincide con el de `requiredApprovals`, pase el `status` a `Approved`.