## Arrays - estructura genérica, funciones genéricas

En TS, no puedo definir que algo es un array, sin decir "un array _de qué_". Es decir, hay que especificar qué tipo deben tener los elementos que voy a agregar.
``` typescript
interface AccountApplication {
    customer: string,
    status: string,
    date?: string,
    requiredApprovals: number
}

function createNewApplication(customer): AccountApplication {
    return { customer, status: 'Pending', date: '2020-04-12', requiredApprovals: 4 }
}

const applications: AccountApplication[] = [
    createNewApplication('Nepomuceno Benítez'), createNewApplication("Fabiola Luzuriaga")
]
```

Esto es muy útil para el _intellisense_, p.ej. si tipeo `applications[0].`, va a proponer los atributos de AccountApplication.  
También para el _chequeo_, probar p.ej. `applications.push("hola")`.


### Tipos en la interface "funcional" de Array

Si un Array puede ser Array "de cualquier cosa' ¿cuál es el tipo del método `filter`? Seguro que tiene una flecha, porque `filter` es una función.  
A su vez, el parámetro de `filter` también es una función. La estructura nos queda así:
``` typescript
filter: ( callbackfn: (value: _1_ => _2_) ) => _3_
```
o sea, es una función que espera una función (el `callback`) por parámetro. El `value` es el parámetro de _esa_ función.

Nos falta definir los "casilleros" 1, 2 y 3.  
Hay uno que es fácil: la función que se le pasa al `filter` debe devolver un booleano.  
Además, de lo que devuelve, sabemos que es un array.
Refinemos un poco el tipo, poniendo lo que sabemos.
``` typescript
filter: ( callbackfn: (value: _1_ => boolean) ) => _3_[]
```
El casillero 1 corresponde a lo que va a recibir la función que se le pasa al `filter`, que es un elemento del array original.  
A su vez, el `filter` devuelve una sublista, o sea, algo del mismo tipo del array original. 

Entonces, los tipos de los casilleros 1 y 3 coinciden, y dependen de "de qué" es el array. Para un array de `AccountApplication` tendremos
``` typescript
Array<AccountApplication>.filter: ( callbackfn: (value: AccountApplication => boolean) ) => AccountApplication[]
```
si fuera un array de números tendríamos
``` typescript
Array<number>.filter: ( callbackfn: (value: number => boolean) ) => number[]
```

Obviamente, no hay muchas definiciones de `filter`, hay una sola. Si tuviera que dar el tipo _de lo que está definido_, o sea sin saber dónde se va a usar, ¿cómo quedaría? Así:

``` typescript
Array<T>.filter: ( callbackfn: (value: T => boolean) ) => T[]
```

O sea, que `filter` es una **función genérica**: aplica a arrays de cualquier tipo, recibe una función que va de valores de _ese_ tipo en `boolean`, y devuelve otro array del _mismo_ tipo.  
La `T` es el nombre que le estamos dando al tipo de los elementos del array. Es una _variable de tipo_.

A su vez, los arrays son **estructuras genéricas**, pueden manejar elementos de cualquier tipo.


### Preguntas y desafíos

Si agrego `createNewApplication(4)` a la definición de `applications` ... ¿qué pasa?  
Armar una expresión de la forma `applications.<algo_que_puede_ser_largo>`, que con este agregado, compila pero da `TypeError`.  
¿Dónde está el problema? Relacionar con lo que vimos sobre los peligros del tipo `any`. Arreglar para que `createNewApplication(4)` no compile.

Mirar el tipo de `filter` como lo muestra VSCode. Estudiar las diferencias con lo que dice acá arriba.  
En particular, dice `unknown` en lugar de `boolean` porque en realidad la función puede devolver _cualquier_ valor, que se va a interpretar como booleano de acuerdo a lo que dijimos sobre valores _truthy_ y _falsy_.  
Si puede ser cualquier cosa ¿por qué `unknown` y no `any`? En TS, `unknown` es un "primo simpático" de `any`, ver detalles en [este artículo que me gustó](https://mariusschulz.com/blog/the-unknown-type-in-typescript).

¿Qué diferencia hay si el tipo de `filter` lo pienso así?
``` typescript
Array<any>.filter: ( callbackfn: (value: any => boolean) ) => any[]
```
**Hint**  
poner `applications.filter((n: number) => n > 5)` y ver qué pasa.  
Idem para `applications.filter(req => req.customer.startsWith("Fabi"))[0].toUpperCase()`.

Escribir el tipo de la función `map`, y _después_{: style="color: Crimson"} verificar en VSCode.

¿Qué tipo tienen todos los arrays, pero solamente los arrays?

Definir el tipo más específico posible para `[3,5,createNewApplication("Perdita Durango")]`. O sea, un tipo que me permita agregarle números, `AccountApplication`, y ninguna otra cosa.

### Una curiosidad

Si tengo un array como en el último ejemplo y me quiero quedar con las `AccountApplication` usando un `filter`, ... podría, pero el tipo de lo que obtengo no va a ser `AccountApplication[]`.

Googleando otra cosa, surgió este caso, y cómo manejarlo usando una sutileza de TS llamada _type guards_.
Les curioses pueden ver [este post en StackOverflow](https://stackoverflow.com/questions/43010737/way-to-tell-typescript-compiler-array-prototype-filter-removes-certain-types-fro) y [esta página de la doc de TS](https://www.typescriptlang.org/docs/handbook/advanced-types.html#type-guards-and-differentiating-types).

## Maps - las variables de tipo se hacen explícitas

Los `Map` son otra estructura de datos que viene con TS (y con JS también). Un `Map` es ... un mapa clave-valor. Están definidos como una clase.  
Los `Map` no tienen la felicidad de los literales como los Arrays, o sea, para crear un `Map` hay que usar `new`:

``` typescript
const myMap = new Map()
```
El tipo que infiere TS para `myMap` es `Map<any,any>`. Los mismos corchetes que al lado de `Array` en la definición del tipo de `filter`. La clase `Map` es una **clase genérica**, en rigor es `Map<K,V>`. Las definiciones genéricas incluyen variables de tipo, la clase Map incluye dos, que son los tipos de clave (`K`) y valor (`V`). Como no especificamos estos tipos, TS los asume como `any`.

¿Cómo especificamos los tipos? Así:
``` typescript
const myMap: Map<number, AccountApplication> = new Map()
```

### Para mirar
Las dos operaciones básicas de un `Map<K,V>` son `set(key,value)` (para agregar un par clave-valor), y `get(key)` (para obtener el valor relacionado con una clave).  
Anotar los tipos para estos métodos, y _después_{: style="color: Crimson"} verificar en VSCode.

La clase `Map` tiene un constructor en el que se le pueden pasar valores, ver [la doc MDN para JS](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/Map). Ver qué tipo infiere si se usa este constructor.


## Definamos un tipo genérico nosotres
Para hacer definiciones genéricas, lo **único** que tenemos que hacer es incluir variables de tipo, así como las indicamos en `Map<K,V>`.

Vamos con un ejemplo rápido: esta definición
``` typescript
interface Pair {
    fst: number,
    snd: string
}
```
nos permite tener pares ... pero donde siempre el primer componente es un número y el segundo un string. 

<div style="font-size: 150%; color: LimeGreen">
(... a partir de acá es para desarrollar ...)
</div>

P.ej.  si tengo
const unPar: Pair = { fst: 4, snd: "hola" }
puedo ... unPar.fst + 4

Acá va `Pair`.

Aclarar la diferencia con poner todo `any`.

### Preguntas

¿Qué pasaría si definimos `Pair` con una sola variable de tipo?

