---
layout: default
---

## Tipado estructural

Miremos estas definiciones

``` typescript
interface Address {
    street: string,
    streetNumber: number
}

const isLongAddress = (addr: Address) => {
    return addr.street.length > 30
}

const isLongAddress2 = (addr: { street: string, streetNumber: number}) => {
    return addr.street.length > 30
}
```

Recordemos lo que dijimos antes, respecto de considerar al tipo como un filtro. Si los vemos así, los tipos `Address` y `{street: string, streetNumber: number}` son **idénticos**: dejan pasar _exactamente_ a los mismos valores.

Incluso si definimos
``` typescript
interface StillAnotherAddress {
    street: string,
    streetNumber: number
}

const stillAnotherIsLongAddress = (addr: StillAnotherAddress) => {
    return addr.street.length > 30
}
```
el nuevo tipo, aunque tiene _distinto nombre_, es equivalente a los anteriores.

Por eso se dice que TS adopta _tipado estructural_: lo que define un tipo está dado por la _estructura_, no por el nombre. Dos definiciones que especifican la misma estructura, son dos nombres para el mismo tipo.  
A veces se usa _duck typing_ como sinónimo para "tipado estructural". Por qué: porque en este esquema de tipado, si un animal camina como pato y hace "cuack", se lo considera pato, por más que no se lo haya especificado como pato. Cuack.  
Para dar un ejemplo del "otro barrio": Java adopta _tipado nominal_, el tipo está dado por su nombre.

[Este artículo](https://medium.com/@KevinBGreene/surviving-the-typescript-ecosystem-interfaces-and-structural-typing-7fcecd54aef5) cuenta la historia. De [este](https://medium.com/redox-techblog/structural-typing-in-typescript-4b89f21d6004) me gusta la frase "the structure _is_ the type".

Tal vez la consecuencia más importante es que se puede hacer esto
``` typescript
const isLong = isLongAddress({street: "Yavi", streetNumber: 338})
```
sin necesidad de especificar un tipo para el argumento. Esto hace la vida más fácil, en particular con código que viene de JS.


### Un par de casos polémicos

Repasemos varios casos en los que el sistema de tipos previene el uso de la función `isLongAddress` con valores incorrectos
``` typescript
isLongAddress("hola")
isLongAddress({brand: 'Nokia', model: '1100', year: 2002})
isLongAddress({street: "Barranqueras"})
isLongAddress({street: 9, streetNumber: 38})
```
incluso, en el último caso, el mensaje de error es específico del atributo `street`. Hasta acá vamos perfecto.

Si queremos forzar, casteando, un valor incompatible, tampoco nos deja
``` typescript
isLongAddress("hola" as Address)
```

Peeeeero los valores que llevan tipo `any` tienen _free pass_ (o al menos eso parece):
``` typescript
isLongAddress("hola" as any)
```
lo acepta y da `TypeError` en ejecución.


**Otra cuestión**:  
¿qué pasa si al argumento le _sobran_ atributos? P.ej. `{ street: "Clorinda", streetNumber: 983, city: "Santa Marta" }`.

Auch, la respuesta es _depende_, de si es un literal o no. Si tenemos:
``` typescript
const extendedAddress = { street: "Clorinda", streetNumber: 983, city: "Santa Marta" }
const x1 = isLongAddress(extendedAddress))
const x2 = isLongAddress({ street: "Clorinda", streetNumber: 983, city: "Santa Marta" })
```
la definición de `x1` compila, pero la de `x2` no. 
El mensaje es claro: `Object literal may only specify known properties ...`.  
¿Por qué tomó esta decisión TS? La verdad que no sé.


### Para jugar
Algunas preguntas
1. ¿Qué pasa si se fuerza usando `as any`, y en realidad el argumento sí tiene la info necesaria para que la función se **ejecute**? ¿Da `TypeError` en ese caso?  
¿Cuál sería un ejemplo para la función `isLongAddress`?

1. ¿Se puede usar casteo para que la definición de `x2` ande? Buscar una forma de casteo que funcione para este caso pero no para los anteriores.
1. ¿Se puede lograr que la definición de `x2` funcione, con alguna técnica distinta al casteo? A mí me salió clonando.
1. Hay una opción de compilador que apunta específicamente a la cuestión de los literales con atributos "de más". ¿Cuál es? Encontrarla y probar.


## Variaciones sobre interfaces
Esta definición
``` typescript
interface ExtendedAddress extends Address {
    readonly isApproximate: boolean,
    postCode: string,
    city?: string
}
```
incluye un popurrí de variantes sobre definiciones de interfaces.

Se puede hacer _herencia de interfaces_, que funciona como uno se imagina.  
Por lo que hablamos antes del tipado estructural, la herencia **no** es necesaria para que las funciones con un parámetro tipado como `Address` acepten valores con el tipo `ExtendedAddress`; esta definición es equivalente a
``` typescript
interface ExtendedAddress {
    street: string,
    streetNumber: number,
    readonly isApproximate: boolean,
    postCode: string,
    city?: string,
}
```

También se pueden definir _atributos `readonly`_, p.ej. si tenemos
``` typescript
const externalSite: ExtendedAddress = {
    isApproximate: false,
    street: "Casabindo",
    streetNumber: 148,
    postCode: "BBB999",
    city: 'La Banda'
}
```
no vale hacer `externalSite.isApproximate = true`.

De los _atributos opcionales_, nada para decir, son lo que uno imagina. Pueden estar o no estar.  
Hay una variante que es dejar una interface _abierta_:
``` typescript
interface FlexibleAddress extends Address {
    [propertyName: string]: any
}
```
esto quiere decir "tiene que estar lo especificado explícitamente, y además puede haber todos los atributos que quieras, sin restricción de nada". P.ej.:
``` typescript
const hugeAddress: FlexibleAddress = {
    street: "Clorinda", streetNumber: 983, city: "Santa Marta", country: "Argentina", 
    planet: "Earth", galaxy: "Milky Way"
}
```

¿Sirve esto para algo más que para los literales? La verdad, no lo sé.

### Preguntitas
Volviendo a lo de atributos readonly, ¿por qué lo siguiente sí anda?
``` typescript
const otherSite = {...externalSite, isApproximate: true}
```
¿se "rompió" el readonly acá?

Y otra: ¿se podrá romper el `readonly` con una interfaz "melliza" sin `readonly`?.

Una sobre opcionales: si un valor `x: ExtendedAddress` no define `city`, y se pide `x.city`, ¿qué se obtiene?

## Tipos función, clases e interfaces
Veamos esta definición
``` typescript
interface AddressAnalyzer {
    priority: number,
    prototypeAddress: Address,
    isLongAddress: (a: Address) => boolean,
    magicAddressChange: (a: Address) => Address
}
``` 
define un objeto con cuatro atributos, un número, un `Address` ... y dos _funciones_.

Veamos formas de usar este tipo.
``` typescript
const checker: AddressAnalyzer = {
    priority: 4,
    prototypeAddress: externalSite,
    isLongAddress,
    magicAddressChange: (a: Address) => { return { street: a.street, streetNumber: 4 }}
}

function makesLong(analyzer: AddressAnalyzer): boolean {
    return analyzer.isLongAddress(analyzer.magicAddressChange(analyzer.prototypeAddress))
}
```

Tenemos un tipo que especifica atributos y funciones ¿a qué se parece? Claro que sí, al formato de una _clase_.  
Las instancias de las clases compatibles con la interface, tienen la interface:
``` typescript
class StandardAnalyzer {
    constructor(public priority: number, public prototypeAddress: Address) { }
    isLongAddress(a: Address): boolean { return a.streetNumber > 100 }
    magicAddressChange(a: Address): Address { return { ...a, streetNumber: a.streetNumber + this.priority }}
}

const x = makesLong(new StandardAnalyzer(5, {street: "Yavi", streetNumber: 338}))
```

### Preguntas
Si en la definición de `StandardAnalyzer` se cambian los `public` por `private`, ¿se rompe algo? ¿Por qué?

Lo mismo, si se agregan métodos a `StandardAnalyzer`.

Teniendo en cuenta que TS adopta tipado estructural ¿cuál es la utilidad del `implements`?


## (uh) la varianza y sus variantes
Concentrémonos en esta definición
``` typescript
type AddressChange = (a: Address) => Address
```
¿Qué información me da saber que una función tiene este tipo? Dos cosas:
1. que la puedo invocar con un argumento `Address`, y no se va a romper.
2. que el valor que va a devolver cumple con el tipo `Address`.

Esto me da la tranquilidad que si la función `fc` tiene el tipo `AddressChange`, la siguiente expresión es correcta
``` typescript
fc({street: 'Yavi', streetNumber: 338}).streetNumber
```

Ahora analicemos esta definición.
``` typescript
function createAddress(a: {street: string}) {
    return {street: a.street, streetNumber: a.street.length * 100, isApproximate: true, postCode: 'AAA555' }
}
```
Si les preguntaran si la función `createAddress` tiene el tipo `AddressChange`, ¿qué dirían?  
La respuesta surge de la información. Esta función se puede invocar con un argumento `Address`, y lo que devuelve cumple con `Address`. O sea ... **oh sí**.

O sea, que cualquier función que define un _parámetro_ de tipo `Address` **o más restringido**, y cuyo _valor de respuesta_ es de tipo `Address` **o más extendido**, se puede decir que tiene el tipo `ChangeAddress`.

Dicho un poco más formal: si `P` extiende `Q`, y `S` extiende `R`, entonces, `Q => S` extiende `P => R`. Acá, pensar "extiende" como el `extends` entre clases.  
En el ejemplo, `P` y `R` son `Address`, `Q` es `{street: string}`, y `S` es `ExtendedAddress`. 

Fíjense que los tipos de respuesta van en la misma dirección que el tipo de la función, mientras que los tipos de parámetro van en la dirección opuesta.  
Por esto se dice que el tipo de una función es _covariante_ respecto de la respuesta, y _contravariante_ respecto del parámetro. Ufffff.

### ¿Dónde se aplica toda esta ciencia?
En el chequeo estático de tipos, claro. 
P.ej. si definimos esta clase

``` typescript
class DifferentAnalyzer {
    constructor(public priority: number, public prototypeAddress: Address) { }
    isLongAddress(a: Address): boolean { return a.streetNumber > 100 }
    magicAddressChange(a: {street: string}) {
        return { street: a.street, streetNumber: a.street.length * this.priority, 
                 isApproximate: true, postCode: 'AAA555' }
    }
}
```

¿compilará esto?
``` typescript
const x = makesLong(new DifferentAnalyzer(5, {street: "Yavi", streetNumber: 338}))
```

Recordemos que el parámetro de `makesLong` está definido como `AddressAnalyzer`, que incluye `magicAddressChange: (a: Address) => Address`. Por todo lo que hablamos, esta línea **sí** va a compilar.


### Todavía falta un detalle

Ahora miremos esta función
``` typescript
const createAddress2: AddressChange = (a: { street: string, streetNumber: number, postCode: string }) => {
    return { street: a.street, streetNumber: a.street.length * a.postCode.length, isApproximate: true, postCode: a.postCode }
}
```
¿Qué decimos de esta, que tiene el tipo `AddressChange`? Deberíamos decir que no, porque no es cierto que funcione con cualquier valor `Address`.

Peeeeero ... TS dice que sí. Por alguna razón que no comprendo, acepta _bivarianza_ respecto de los parámetros, cuando vimos que lo seguro es aceptar solamente _contravarianza_.

Y sí, se rompe: si cambiamos la definición de `magicAddressChange` por esta, y evaluamos `makesLong`, da `TypeError` en ejecución (pensar bien por qué).

Para <s>emparchar</s>manejar esta cuestión está la opción de compilador `strictFunctionTypes`. La descripción en el [sitio de TS](https://www.typescriptlang.org/docs/handbook/compiler-options.html) 
> Disable bivariant parameter checking for function types.

es bien clarita ... si uno sabe qué es "bivariant".

### Y si todavía quedan ganas ... algunas preguntas

La primera es una clásica del estudio de tipos: ¿qué pasa con la varianza y la covarianza si _el parámetro_, y/o _el valor de retorno_, son de tipo _función_.  
**Hint** cómo lo pienso yo: como si fuera multiplicación de signos, menos por menos es más, etc..

Una más "terrenal": si cambiamos la definición de `createAddress2` por
``` typescript
const createAddress2: AddressChange = (a: { street: string, streetNumber: number, postCode: string }) => {
    return { street: a.street, streetNumber: a.street.length * 2, isApproximate: true, postCode: a.postCode }
}
```
(el cambio está en `streetNumber`, cambió `a.postCode.length` por `2`)  
¿anda `makesLong`? ¿Qué devuelve `createAddress2({street: "Yavi", streetNumber: 338})`? ¿Cómo se puede explicar este comportamiento?

La última: quiero definir una función `createAddressFromExtended`, con el mismo código que `createAddress`, pero que acepte solamente argumentos que tengan el tipo `ExtendedAddress`.  
¿Cómo definirla sin repetir código ni invocar a la función `createAddress`?  
Esta función ¿podrá usarse como `magicAddressChange` de un `AddressAnalyzer`?

