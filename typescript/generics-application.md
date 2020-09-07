---
layout: default
---

## Pedidos genéricos

Este código
``` typescript
enum Status { Pending, Analysing, Accepted, Rejected }

interface Request<T> {
    resource: T,
    status: Status,
    date?: string,
    requiredApprovals?: number
}
```

define un tipo genérico `Request`, que modela un pedido que se hace de un determinado recurso.  
El tipo `Request` define los datos propios del pedido, más un recurso del cual no conoce nada.

Se está usando la misma forma de tipo genérico que en `Pair`, para un modelo más cercano al negocio.

A partir de la definición de dos tipos de recurso
``` typescript
enum Currency { ARS = "ARS", USD = "USD", EUR = "EUR" }
enum CardIssuer { Visa, AmEx, Mastercard }

interface GenericAccount {
    customer: string,
    currency: Currency
}

interface Card {
    issuer: CardIssuer,
    creditLimit: number
}
```

podemos construir objetos y funciones relacionadas con pedidos, que preservan los tipos de recurso.
``` typescript
const accountRequest: Request<GenericAccount> = { 
    resource: {customer: "Juana Molina", currency: Currency.ARS},
    status: Status.Pending,
    date: "2020-02-21"
}

const visaRequest: Request<Card> = {
    resource: {issuer: CardIssuer.Visa, creditLimit: 300000},
    status: Status.Pending,
    requiredApprovals: 8
}

type AccountRequest = Request<GenericAccount>

function isLocal(req: AccountRequest) { return req.resource.currency === Currency.ARS }

function makeMoreStrict<T>(req: Request<T>) { 
    if (req.requiredApprovals) {
        req.requiredApprovals++
    }
}
```
Dos detalles en este código
- el uso de `type` para crear una abreviatura de un tipo de nombre largo.
- el tratamiento de un atributo opcional en `makeMoreStrict`.



## Para explorar

### Función no genérica sobre tipo genérico
En rigor, no es necesario que `makeMoreStrict` sea una función genérica. Pero `Request` sí es genérico. ¿Qué se podría poner como parámetro en la definición del atributo? Relacionar con el tipo de retorno de la función.

### Uso de generics acotados
Definir una función `isWeak` que recibe un `AccountRequest`, y devuelve `true` si: el valor de `requiredApprovals`  es menor a 3 o no está definido, la moneda es dólares, y el nombre del cliente empieza con minúscula.

Cambiar la definición de `isLocal` para que sea válida para los pedidos de cualquier recurso que tenga una `currency`.

### Incorporamos clases
Consideremos estas clases 
``` typescript
class BankAccount {
    constructor(public customer: string, public currency: Currency) {}
    description(): string { return `Account owned by ${this.customer}` }
}

class Credit {
    constructor(public customer: string, public amount: number, public rate: number) {}
    description(): string { 
        return `Credit of ${this.amount} given to ${this.customer}` 
    }
    get currency() { return Currency.ARS }
}
```
Algunas preguntas:
- ¿Puede usarse la función `isWeak` para los pedidos de instancias de `BankAccount`?
- Si se cambia `isLocal` como se indica en el desafío previo ¿serviría para instancias de `Credit`?

### Las ventajas de poner los tipos
Definir varios objetos que cumplan con la interface `Request<GenericAccount>`, verificar que pueden usar las funciones `isWeak` e `isLocal`.  
Ahora, cambiar la interface `Request`, p.ej. en el nombre del atributo de `requiredApprovals` sacar la `s` final. Ver qué pasa si se especifica el tipo de los objetos, o sea
``` typescript
const juanaRequest: Request<GenericAccount> = {}
```
y qué pasa si no se especifica.

### Listas con elementos genéricos
Definir la función `requestsInYear`, que recibe una lista de `Request` y un año, y devuelve una nueva lista con los pedidos que sean de ese año.

Lograr que si se la invoca con una lista de pedidos homogéneas, o sea que sean todas del mismo tipo de recurso (p.ej. una lista de `Request<Credit>`), el resultado tipe como una lista de pedidos de ese mismo tipo.  
¿Qué pasa si la lista no es homogénea, cómo usar la función, qué puedo hacer con lo que devuelve?

### Uno más
Definir la función `pendingDescriptions()`, que recibe una lista de `Request` a cuyo recurso se le puede pedir la `description()` (como es el caso de `BankAccount` y `Credit`), y devuelve la lista de las `description()` de los (recursos de los) pedidos cuyo estado es `Pending`.

Definir el tipo preciso para el parámetro. 

### Ultra desafío
Variante sobre lo anterior: definir una función que dada una `Card`, devuelva otra con las mismas características, más una función  
``description() { return `Tarjeta ${this.cardIssuer} con límite ${this.creditLimit}` }``  

de forma tal que se pueda usar `pendingDescriptions()` sobre una lista de pedidos de `Card` transformadas de acuerdo a la función anterior.

Yo estuve un rato ... la versión que más me convenció usa `Object.create()`.

