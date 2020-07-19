# Arranquemos con Mongoose
En esta página, daremos una brevísima introducción a Mongoose. 
El obejtivo es presentar un conjunto mínimo de elementos que sean suficientes para comenzar a interactuar con una base MongoDB desde un programa.  
En un primer momento usaremos JavaScript como lenguaje y haremos pequeños scripts que ejecutaremos usando Node. Luego de adquiridos los elementos básicos, veremos cómo utilizarlos desde una aplicación NestJS en TypeScript.

En esta página, y en las que siguen, nos limitaremos a presentar los elementos que utilicemos. Los detalles se pueden consultar en la [documentación de Mongoose](https://mongoosejs.com/docs/). Incluiremos los links específicos dentro de esta documentación, que incluyan información extendida de distintos temas que iremos tocando.

La interface de Mongoose puede explicarse a partir de cuatro conceptos
1. **conexión**: acceso a una BD.
1. **esquema**: especificación de la forma de un documento.
1. **modelo**: acceso a una colección.
1. **operaciones sobre datos**: búsquedas / altas / modificaciones / bajas.

Veamos brevemente cada uno.


## Conexión
Es el objeto mediante el cual se accede a una base de datos. Es un concepto común en las [librerías de acceso a BD](./librerias.md).

La especificación de la BD a acceder se hace mediante el _string de conexión_ de la base, el concepto de [string de conexión](https://en.wikipedia.org/wiki/Connection_string) esta muy extendido en BD de distintos modelos.  
Actualmente, estos string toman la forma de una URL.

En principio, el mismo objeto `mongoose` que importamos es el que representa la conexión. Por eso no es necesario "guardar" el resultado de la operación de conexión, que tiene esta forma.
``` javascript
await mongoose.connect(
    'mongodb://<host>/<nombre-de-la-base>', 
    { useNewUrlParser: true, useUnifiedTopology: true }
);
```
el segundo parámetro son las [opciones de conexión](https://mongoosejs.com/docs/api/mongoose.html#mongoose_Mongoose-connect). Reconozco que motivación de las dos que están puestas es que no salten warnings al ejecutar los scripts.

Una conexión es un recurso que la BD pone disponible para nuestro programa, y por lo general las bases tienen una cantidad limitada de conexiones. Por lo tanto, al terminar nuestro script, seremos gentiles y nos _desconectaremos_.
``` javascript
await mongoose.disconnect();
```

Para más detalles, se puede consultar [la página de la documentación que se refiere a conexiones](https://mongoosejs.com/docs/connections.html).

Si en nuestro programa necesitáramos acceder a varias bases, tenemos que pasar al esquema que se describe [en la documentación](https://mongoosejs.com/docs/connections.html#multiple_connections).


## Esquema
Un esquema es la _especificación_ de la forma que van a tener documentos que se incluyan en la base.  
Notar que todavía no hablamos de _colecciones_: sólo estamos especificando qué _atributos_ debe tener un _documento_.  
Para cada atributo se define el tipo de dato, y eventualmente configuraciones adicionales. Varias de estas configuraciones tienen que ver con validaciones. 

Esta es la defnición de un esquema con cuatro atributos sencillos.
``` javascript
const accountRequestSchema = new mongoose.Schema({
    customer: { type: String, required: true },
    status: { type: String, enum: ['Pending', 'Analysing', 'Accepted', 'Rejected'] },
    date: Number,
    requiredApprovals: Number
})
```

> **Atención**  
> Es importante "guardar" una referencia al esquema creado, los modelos se crean a partir del esquema.

La esepecificación
``` javascript
    date: Number,
```
es simplemente una forma abreviada para
``` javascript
    date: { type: Number },
```
que puede usarse si no se necesita ninguna configuración especial para el atributo.

Los atributos `required` y `enum` dentro de la definición de los atributos `customer` y `status` respectivamente, son ejemplos de validaciones.
Cualquier intento de agregar un documento que no cumpla con las validaciones, se rechazará saliendo con excepción.  
La lista completa de tipos y configuraciones para un atributo se encuentran en la [guía sobre SchemaTypes de la documentación](https://mongoosejs.com/docs/schematypes.html).

En esta etapa, nos limitaremos a atributos de tipos de datos simples; más adelante en la capacitación aparecerán atributos compuestos que involucren arrays y/u objetos.

Además de la definición de los atributos, se pueden incluir más funcionalidades dentro de un esquema, que permiten simplificar transformaciones, agregar atributos calculados, etc.. Veremos algunas más adelante en esta sección. Para un detalle completo puede consultarse [la guía sobre esquemas en la documentación](https://mongoosejs.com/docs/guide.html).


## Modelos
Un modelo permite el acceso a una colección. 
``` javascript
const accountRequestModel = mongoose.model('AccountRequest', accountRequestSchema);
```
se especifican: el nombre de la colección, y el esquema que van a tener los documentos en esta colección.  
En rigor, para armar el nombre de la colección, Mongoose pasa el nombre indicado a minúscula, y lo lleva a plural. En este ejemplo, el modelo va a acceder a la colección `acccountrequests`.

> **Un detalle**  
> Aquí `mongoose` no se refiere a la librería, sino a la _conexión_. Si tuviéramos que manejar múltiples conexiones, en lugar de `mongoose` iría una referencia a la conexión. 

Las operaciones sobre una colección se le solicitan al modelo correspondiente.

Más detalles en la [guía sobre modelos en la documentación](https://mongoosejs.com/docs/models.html).


## Operaciones
Como dijimos, para todas las _operaciones_ interviene el modelo de la colección correspondiente. 
Dos aclaraciones antes de revisar operaciones individuales
*  Es importante entender que los objetos involucrados en una operación son _documentos_, que son objetos propios de Mongoose. La guía incluye una [página sobre documentos](https://mongoosejs.com/docs/documents.html).
* Las operaciones son (obviamente) _asincrónicas_. Por lo tanto, debemos usar lo que vimos sobre `async/await`.


### Alta de un documento
Para agregar un nuevo documento (o sea, hacer un insert), una forma (entre varias que permite Mongoose) incluye dos pasos. Primero se crea un documento con los datos deseados. Después se le indica `save` a este nuevo documento, que es un objeto Mongoose.

Con un ejemplo:
``` javascript
const requestData = { customer: 'Pedro Navaja', status: 'Pending', requiredApprovals: 5 };
const req = new accountRequestModel(requestData);
await req.save();
```

### Búsqueda de documentos
Para esto usamos la operación `find` incluida en los modelos. 
``` javascript
const allTheRequests = await accountRequestModel.find();
```

Lo que vamos a obtener es un array de _documentos_. A un documento se le pueden pedir los valores de los atributos como si fuera un objeto sencillo.
``` javascript
allTheRequests[0].status 
```

pero también tiene comportamiento propio de Mongoose. En particular, nos va a servir para la modificación (un poco más abajo).

Con el `find` sin parámetros, obtenemos **todos** los documentos de la colección. Obviamente se pueden aplicar filtros, que veremos en una página aparte.


### Modificación de documentos
Se hace simplemente modificando los valores de los atributos de un documento, e invocando `save` sobre el mismo.
``` javascript
firstRequest = allTheRequests[0]
firstResult.requiredApprovals = 21
await firstResult.save()
```
(¡no olvidar el `await`!)

> **Pregunta**  
> ¿Se pueden hacer operaciones _masivas_, p.ej. cambiar el valor de un atributo a un grupo de documentos en una sola operación?  
> Sí ... y lo veremos en una etapa posterior.


## Algunas pruebas para hacer
Intentar el agregado de documentos que no cumplen con las validaciones definidas en el esquema.

Crear un documento a partir de un objeto que incluya atributos no definidos en el esquema, insertarlo, ver qué pasa.

Ver en la documentación [sobre modelos](https://mongoosejs.com/docs/api/model.html) cómo hacer para borrar un documento, y probar. Verán que hay varias opciones, creo que la recomendada es `findOneAndDelete`.
