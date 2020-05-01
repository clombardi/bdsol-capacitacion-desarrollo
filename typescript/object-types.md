## Tipos de los objetos
Si ponemos 
``` typescript
const anApplication = {
    customer: '99775533112',
    status: 'Rejected',
    date: '2020-03-04',
    requiredApprovals: 2
}
```
y nos paramos arriba de `anApplication` ¿qué tipo indica?
``` typescript
{
    customer: string;
    status: string;
    date: string;
    requiredApprovals: number;
}
```
... un tipo sin nombre. ¿Para qué sirve?

Para lo que más sirve (en mi opinión) el tipado en TS: para el intellisense.   
Probemos poner
``` typescript
anApplication.
```
¡magia! propone los atributos.


### ¿Qué es un tipo?
Seguro que no es una especificación de qué forma tiene el objeto en memoria. Este es el concepto de tipo de p.ej. C (y de ahí lo de `int32`, `int64`, `uint`, etc.), que **no aplica** a TS.  

------
**Nota**{: style="color: SteelBlue"}:  
De hecho a Java tampoco, la Java Language Specification indica explícitamente que _no podés saber_ cómo organiza la memoria la VM.

------

Pensemos, si un tipo fuera la especificación del formato en memoria ¿qué sentido tendría el tipo `(number | string)`.

Un tipo es una especificación de _un conjunto de valores_. Dado un tipo, hay algunos valores que tienen el tipo (notar que no digo "_son_ del tipo", digo "_tienen_ el tipo") y otros que no.

Muchos lugares en los que se especifican tipos pueden verse como un _filtro_: algunos valores van a pasar, otros no.

------
**Nota**{: style="color: SteelBlue"}:  
El `typeof` nos da una clasificación burdísima, p.ej. a todos los `object` les asigna el mismo tipo.  
Por otra parte ... el `typeof` podemos usarlo para saber _cómo tratar_ a un valor, pero no si el lenguaje _lo va a aceptar o no_.  
Para esto último (la aceptación) se necesita que el lenguaje haga _chequeo **estático** de tipos_. JS no lo hace, TS sí.  
Lo que hacemos con el `typeof` es _chequeo **dinámico** de tipos_. Esto sí lo permiten hacer tanto JS como TS.  
Groseramente: "estático" = antes de ejecutar, "dinámico" = durante la ejecución. Los linters también son estáticos.

------

... ok, vamos a un caso concreto.



### Parámetro de una función

Digamos que una solicitud es exigente si requiere más de tres aprobaciones. Fácil
``` typescript
function isDemanding(appli) { return appli.requiredApprovals > 3 }
```

como vimos apenas empezamos a hablar de tipos, si no digo nada toma `any` como tipo del parámetro. 

El quick fix propone el tipo `{ requiredApprovals: number }`.

Pregunta: `anApplication` ¿_tiene_ ese tipo? Sí, porque esta especificación sólo indica "tiene el atributo", puede tener ese atributo y muchos más.

------
**Nota**{: style="color: SteelBlue"}:  
Cualquier tipo "con llaves" sólo lo pueden tener objetos, o sea valores cuyo `typeof` sea `object`.

------


### Interfaces

Uno podría decidir que como especificación es demasiado relajada, más que `any` claro, pero podría ser más estricta.  
P.ej. podría querer que la función sólo aceptara applications, o sea objetos que tengan los cuatro atributos. 
¿Qué hago, pongo 
``` typescript
function isDemanding(appli: { 
    customer: string;
    status: string;
    date: string;
    requiredApprovals: number;
}) { return appli.requiredApprovals > 3 }
```
? Por supuesto que anda, pero es medio largo. Además, si después resulta que las application tuvieran un atributo más, tengo que peinar todas las definiciones de este estilo ... no queremos.

Acá nos ayudan las **interfaces**
``` typescript
interface AccountApplication {
    customer: string,
    status: string,
    date: string,
    requiredApprovals: number
}

function isDemanding(appli: AccountApplication) { return appli.requiredApprovals > 3 }
```

Más sobre interfaces en la siguiente sección.


### Repaso de referencias y casteo
Ahora definimos
``` typescript
const newApplication = anApplication
```
Toma el mismo tipo. De hecho, _es el mismo valor_, estamos definiendo una segunda _referencia_ al mismo objeto. Probemos cambiando p.ej. `newApplication.customer` y veamos cómo queda `anApplication`.

¿Y qué pasa si ahora hacemos?

``` typescript
const newApplication: any = anApplication
```
Separemos en dos cuestiones

1. **como referencias** son dos referencias al mismo objeto. 
1. **como tipos** son distintos,  a `newApplication` puedo "hacerle cualquier cosa". P.ej. `newApplication.requiredApprovals = "hola"`.

Tal vez ayude pensar esto: el primer aspecto sí impacta en el JS resultante, el segundo no. 

Esta definición es una variante del _casteo_, que en TS se expresa con `as`:  
`(firstApplication as any).requiredApprovals = "hola"`  
hay otra sintaxis que es poner el tipo entre `<>` :  
`(<any>firstApplication).requiredApprovals = "hola"` 

------
**Nota**{: style="color: SteelBlue"}:  
El casteo en TS _no hace conversión_, está sólo para que el compilador "deje pasar". Ver [esta notita](https://techformist.com/type-casting-typescript/).

------




### Para jugar

Empecemos por algo rápido:  
¿te acordás cómo hacer que `newApplication` sea un clon de `anAppplication`? Hacelo y fijate qué tipo infiere.

Definamos ahora una variante un poco más compleja de `AccountApplication`
``` typescript
interface AccountApplicationVariant {
    customer: { name: string, fiscalId: string, assetsAmount: number }
    status: string;
    date: string;
    requiredApprovals: number;
}
```
Definir una instancia de este tipo, y hacerle un clon. Cambiar el `clon.customer.name`. ¿Qué pasó con la instancia original? ¿Cómo se explica? Relacionar con lo que hablamos de "shallow copy" y "deep copy".  

Armar una función que devuelva un "deep copy" de una `AccountApplicationVariant`. Aplica el plus de resolverlo con una expresión.

------
**Nota**{: style="color: SteelBlue"}:  
Si el deep clone es una operación habitual en mi aplicación, puedo usar una librería. Buscar `deep clone` en npm.

------

Otra: ¿cómo hacer que la función `isDemandingApplication` acepte cualquiera de las dos variantes de `AccountApplication`? ¿Y si quiero que el criterio cambie de acuerdo a si es `AccountApplication` o `AccountApplicationVariant`?  
Respecto de esto pensar ¿se puede hacer un análogo del `instanceof` de Java, en TS?

