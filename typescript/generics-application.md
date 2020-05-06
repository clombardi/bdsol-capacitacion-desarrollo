## Pedidos genéricos

Este código
``` typescript
enum ApplicationStatus { Pending, Analysing, Accepted, Rejected }

interface Application<T> {
    resource: T,
    status: ApplicationStatus,
    date?: string,
    requiredApprovals?: number
}
```

define un tipo genérico `Application`, que modela un pedido que se hace de un determinado recurso.  
El tipo `Application` define los datos propios del pedido, más un recurso del cual no maneja información.

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

podemos cconstruir objetos y funciones
``` typescript
const accountApplication: Application<GenericAccount> = { 
    resource: {customer: "Juana Molina", currency: Currency.ARS},
    status: ApplicationStatus.Pending,
    date: "2020-02-21"
}

const visaApplication: Application<Card> = {
    resource: {issuer: CardIssuer.Visa, creditLimit: 300000},
    status: ApplicationStatus.Pending,
    requiredApprovals: 8
}

type AccountApplication = Application<GenericAccount>

function isLocal(req: AccountApplication) { return req.resource.currency === Currency.ARS }

function makeMoreStrict<T>(req: Application<T>) { 
    if (req.requiredApprovals) {
        req.requiredApprovals++
    }
}
```
Dos detalles en este código
- el uso de `type` para crear una abreviatura de un tipo de nombre largo.
- el tratamiento de un atributo opcional en `makeMoreStrict`.



### Para explorar
En rigor, no es necesario que `makeMoreStrict` sea una función genérica. Pero `Application` sí es genérico. ¿Qué se podría poner como parámetro en la definición del atributo?

Definir una función `isWeak` que recibe un `AccountApplication`, y devuelve `true` si: el valor de `requiredApprovals`  es menor a 3 o no está definido, la moneda es dólares, y el nombre del cliente empieza con minúscula.

Cambiar la definición de `isLocal` para que sea válida para los pedidos de cualquier recurso que tenga una `currency`.

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

... lista de pedidos con description(), función que da las descripciones de los pedidos pendientes, función que devuelve versiones inyectadas con description() a pedidos de `Card` ...
... y revisar el script ...