## Operaciones externas
Llamemos _operación externa_ a cualquier operación que se invoca dentro de una VM JS (que puede ser p.ej. una instancia de Node o un browser), y que se resuelve fuera de esa VM.

En un _backend_, algunos casos de operaciones externas son:
- invocación de otro microservicio en un entorno de microservicios
- consultas u operaciones sobre una base de datos
- invocación a funcionalidad en otro sistema (p.ej. un core de negocio, MercadoPago, etc.)

En un _frontend_, toda llamada a backend es una operación externa.


## Procesamiento asincrónico
Cualquier llamada a una operación externa, debe manejar el hecho de que el resultado de esa operación va a llegar sólo eventualmente.

En lo que sigue, usamos una invocación HTTP como ejemplo de operación externa. Donde dice `axios.get(<url>)`, bien podría decir p.ej. `<repository>.findOne(<conditions>)`, o una función desarrollada por nosotros que invoca a alguno de estos.

### async-await - la que nos sabemos todos
La experiencia nos dice que este código
``` javascript
function someBusinessCode() {
    const response = axios.get("<url>")
    return doSomething(response.data)
}
```
falla, porque `response.data` es `undefined`. Un `console.log(response)` nos da esto
```
Promise { <pending> }
``` 
¿qué quiere decir esto? Que el resultado de la operación externa es una **promesa**{: style="color: Crimson"}.  
Una promesa ... ¿de qué? De que _en algún momento_ va a llegar el valor. 

Volviendo a la experiencia, sabemos que a este código le faltan un `await` y un `async`.
``` javascript
async function someBusinessData() {
    const response = await axios.get("<url>")
    return doSomething(response.data)
}
```
Con el `await`, estamos forzando a que la función espere que llegue el valor de la operación externa; recién en ese momento se evalúa la línea siguiente. 
Ahora `response` es lo que estábamos esperando:
``` javascript
{
    status: 200,
    headers: { ... },
    data: { ... },
    ...
}
```
A su vez, toda función donde se hace un `await`, debe ser "marcada" como `async`, para que quienes la invoquen sepan que, a su vez, deben hacer `await` para esperar su resultado.
``` javascript
const businessData = await someBusinessData()
/* ... etc ... */
```

------
**Nota**{: style="color: SteelBlue"}:  
En Node, es especialmente crítico que las llamadas externas no sean bloqueantes.  
Una invocación bloqueande puede frenar **todo** el funcionamiento de una VM, dado que ni JS ni Node manejan multithreading en forma nativa.   
Al poner `await`, o usar promesas, habilitamos a la VM de Node a atender otras consultas hasta que llegue el resultado externo.

------


### Promesas - el "nombre verdadero" del async-await
Estos son dos métodos de un servicio NestJS
``` typescript
  async addAddress(personId: string, address: NewAddressRequestDto): Promise<Person> {
    const person = await this.personRepository.findOneOrFail(personId, { relations: ['addresses'] })
    const newAddress = await this.addressRepository.save(address)
    person.addresses.push(newAddress)
    return this.personRepository.save(person)
  }

  findAddressesByPerson(personId: string): Promise<Address[]> {
    return this.addressRepository.find({ person: { id: toNumber(personId) } })
  }
```
Concentrémonos en el **tipo de respuesta** de estos métodos: son `Promise<...tipo...>`. 

El `findOneOrFail` también responde `Promise`:
![Tipo de respuesta de `repo.find` es Promise](./images/find-type-promise.jpg)
y lo mismo pasa con `axios.get`.  
![Tipo de respuesta de `axios.get` es Promise](./images/axios-type-promise.jpg)
Estas `Promise` son _las mismas_ que las del `console.log` de arriba, cuando no habíamos puesto el `await`. 

**Notita**:
`Promise` es un tipo genérico.

Volviendo al ejemplo de la sección anterior, podemos implementarlo usando `Promise` así
``` javascript
function someBusinessData() {
    return axios.get("<url>").then(response => doSomething(response.data))
}
```
notar que la función ya no necesita estar "marcada" con `async`.  
Las Promises son objetos a que se les puede indicar `then` con una _continuación_[^1], o sea, una función que se evalúa cuando la promesa se resuelve, o sea, cuando llega el resultado de la operación externa (en el ejemplo, cuando llega el valor del `axios.get`).  
A su vez, _el `then` también devuelve una promesa_, por lo tanto `someBusinessData`  está devolviendo esa promesa.


Por lo tanto, quienes invoquen a esta función, pueden seguir un patrón parecido ...
``` javascript
someBusinessData().then(businessData => /* ... etc ... */)
```
... pero **también pueden mantener la sintaxis con el `await`**{: style="color: Crimson"}.  

O sea: por más que la función se haya armado devolviendo explícitamente una `Promise`, y no tenga `async`, se la puede invocar así:
``` javascript
const businessData = await someBusinessData()
/* ... etc ... */
```


Cualquier función que devuelve una `Promise` puede ser llamada usando `await`, no es necesario marcarla con `async`.  

En particular, este es el caso del segundo método en el servicio Nest. El `find` devuelve una `Promise`, el método del servicio se limita a devolver la misma `Promise`. No hace falta poner `async`.  
En el otro método del servicio, el `async` es necesario para que compile, porque se hacen `await` adentro. Esto es así en JS, y lo hereda TS.

> En rigor, el dúo `async/await` es (al menos hasta donde sé) un syntax sugar para no tener que andar haciendo `then` y pensando en promesas todo el tiempo.  
En JS, el uso `async/await` habilita a que las `Promise` no se mencionen para nada en el código. En TS las seguimos viendo, en el valor de retorno de las funciones asincrónicas.

**Pensando en tipos**:  
si una función devuelve `Promise<Algo>`, el `await` de la llamada "desempaqueta" el `Promise`. Por eso si el tipo de retorno de `someBusinessData()` es `Promise<SomeData>`, el de `await someBusinessData()` va a ser `SomeData`.


### Secuencias de operaciones asincrónicas, errores
Si una función realiza varias operaciones asincrónicas le tiene que poner `await` a cada una
``` javascript
async function otheBusinessData() {
    const response = await axios.get("<other_url>")
    const second_response = await doSomethingAsync(response.data)
    return /* other stuff */
}
```

Si lo implementamos usando `Promises`, aprovechamos que el `then` devuelve una promesa para hacer el encadenamiento de `then`s..
``` javascript
function otheBusinessData() {
    return axios.get("<other_url>")
        .then(response => doSomethingAsync(response.data))
        .then(second_response => /* other stuff */)
}
```
Acá es _muy_ importante no olvidarse del `return` de adelante, porque hay que devolver la promesa ... que genera el _último_ then. Y se empieza a ver cómo la sintaxis `async-await` nos simplifica la vida.

Para manejar errores, usando `async-await` nos apoyamos en la vieja y querida estructura `try-catch`.
``` javascript
async function otheBusinessData() {
    try {
        const response = await axios.get("<other url>")
        const second_response = await doSomethingAsync(response.data)
        return /* other stuff */
    } catch (err) {
        /* error handling */
    }
}
```

Si usamos promesas, el `try-catch` **no funciona**. Hay que decirle `.catch` al objeto `Promise`, esto me devuelve otra `Promise` que se puede seguir encadenando.
``` javascript
function otheBusinessData() {
    return axios.get("<other_url>")
        .then(response => doSomethingAsync(response.data))
        .then(second_response => /* other stuff */)
        .catch(err => /* error handling */)
}
```

## Muy lindo todo esto ... ¿sirve para algo?
En principio, se podría pensar que al agregarse la sintaxis `async/await`, las `Promise` quedaron como cultura general.

Creo que igual es útil entender el concepto de promesa, hasta donde vimos por dos razones:
1. entender por qué en TS el tipo de retorno de una función asincrónica es `Promise<SomeData>`, de dónde viene eso y por qué hay que hacerlo así. También, que el `await` "desempaqueta" la `Promise` y nos deja un valor de tipo `SomeData`.
1. saber que si tengo una función legacy que devuelve una `Promise` explícitamente, aunque no esté definida como `async` la puedo usar con `await`.

Hay otro escenario, en el que nos va a ser útil usar `Promise` en forma explícita. Keep in touch.

<br/><br>

-----
[^1]: ¿por qué "continuación" y no "callback"?  
En caso de encadenamiento, si trabajo con callbacks es el primer callback que llama al segundo, en cambio con continuaciones es el mecanismo que las maneja (en este caso las promesas) el que maneja la secuencia.

El callback es un valor adicional que se le pasa a la función asincrónica. El ejemplo con dos operaciones asincrónicas, en callback quedaría así
``` javascript
function otheBusinessData() {
    return axios.get(
        "<other_url>", 
        response => doSomethingAsync(
            response.data, 
            second_response => /* other stuff */
        )
    )
}
```

**Notita histórica**:  
nótese cómo el código se va corriendo hacia la derecha a medida que encadenamos operaciones asincrónicas. Esto es lo que se conoce como "callback hell". 
Uno de los argumentos de venta de las promesas cuando aparecieron en el mundo JS es evitar el "callback hell". 