---
layout: default
---

## Object literals

Compañeros de ruta de todo JavaScriptero.
``` javascript
let windowSpec = {
    height: 200,
    width: 150
} 
```

Puedo pedir un atributo, puedo cambiar valores. En principio son abiertos, puedo agregar lo que quiera. Si pido un atributo que no tiene definido, obtengo `undefined`. También puedo pensar en un objeto como un mapa (... que es, creo, como lo piensa JavaScript ...), por lo tanto pedirle los keys y values.
``` javascript
> windowSpec.height
200
> windowSpec.height = 100
100
> windowSpec.color = 'blue'
'blue'
> windowSpec
{ height: 100, width: 150, color: 'blue' }
> windowSpec.preferredPhilosopher
undefined
> Object.keys(windowSpec)
['height', 'width', 'color']
> Object.values(windowSpec)
[ 100, 150, 'blue' ]
```

También está la notación `<objeto>[<atributo>]` que permite obtener un atributo sin fijar el nombre
``` javascript
let attrNamePrefix = 'widt'
windowSpec[attrName + 'h']
```

### Referencias

Si hago
``` javascript
let windowSpec = { height: 200, width: 150 }
let otherSpec = windowSpec 
const thirdSpec = windowSpec
```

los tres identificadores hacen referencia _al mismo objeto_. 
Probar qué pasa con cualquiera de los tres si después se hace

``` javascript
windowSpec.height = 100
```

(_pregunta_: ¿qué pasa si hago `windowSpec = 100`, también cambian todos?)

Distinta es la cosa si hacemos
``` javascript
let otherSpec = {...windowSpec}
```
porque se está generando un _clon_ de `windowSpec`. Parece que los "tres-puntos" [tienen una variedad de usos](https://dev.to/blacksonic/the-tale-of-three-dots-in-javascript-4287). Este es un syntax sugar para `Object.assign()`, o sea, un _shallow copy_. Ver la [diferencia entre shallow copy y deep copy](https://levelup.gitconnected.com/difference-between-shallow-and-deep-copy-c0a968e89c44).

### Identidad e igualdad 
La diferencia entre referencias-al-mismo-objeto y clones, se puede testear con los operadores `===` y `==`. El primero sólo da `true` para referencias-al-mismo-objeto, el segundo también da `true` para clones.  
Probar `windowSpec === otherSpec` y `windowSpec == otherSpec` con las dos definiciones de `otherSpec` que dimos.

### Para ir cerrando
Los "tres-puntos" permiten mergear varios objetos, y también agregar/modificar valores.

``` javascript
let point = {x:8, y:12, z:-4}
let otherSpec = {...windowSpec, ...point, width:75, borderWidth:4}
```

Terminamos esta parte mostrando una variante sintácticamente parecida, pero con un efecto muy distinto. Es esto
``` javascript
let otherSpec = { windowSpec }
```
que es simplemente una abreviatura para `{ windowSpec: windowSpec }`.

(_pregunta_: si con esta definición cambio `windowSpec.height` ¿cambia algo en `otherSpec`?)

------
**Comentario**{: style="color: SteelBlue"}:  
Lo de las referencias compartidas y los "tres-puntos" corre también para _arrays_.  
De hecho ... **los arrays son objetos**, probar `Object.keys(['a', 'b', 'c'])`, notar la similitud entre `windowSpec['width']` y `someArray[1]`.

------

### Una duda y varios desafíos

#### Lo que devuelve una función
Si defino
``` javascript
function theSpec() {
  return { height: 200, width: 150 } 
}
```
e invoco varias veces esta función ¿obtengo siempre el mismo objeto, o cada vez un clon distinto?

Si es siempre el mismo ¿cómo hacer para que devuelva clones?

Si son clones ¿cómo hacer para que devuelva siempre el mismo?


#### Testeando los límites del _shallow copy_
Armar un objeto `x` tal que si defino `y = {...x}` y hago algún cambio "dentro" de `x` (o sea, hago `x.<cosas> = <nuevoValor>`), se modifica también algo en `y`.  

#### Revolviendo un objeto en forma genérica
Definir una función que dado un objeto, devuelva los keys cuyo value asociado es 0. P.ej.
``` javascript
> keysForZero({x:4,y:0,z:1,w:9,f:0})
['y', 'f']
```
A mí me salió con una expresión, o sea 
``` javascript
function keysForZero(obj) {
    return <expresion>
}
```
usando `Object.entries` y métodos de array.  
Un detalle: un array de dos posiciones se puede desarmar con un pattern de la forma `[a,b]`.

#### Rearmando un objeto en forma genérica
Definir una función que, dado un objeto cuyos valores son todos numéricos, devuelve un objeto con las mismas keys, y cada value el doble del value original. P.ej.
``` javascript
> doubledObject({x:4,y:0,z:1,w:9,f:0})
{x:8,y:0,z:2,w:18,f:0}
```
Otra vez, más desafío si sale con una expresión.
A mí me salió usando `Object.values` más estas dos cosas:

- si tengo p.ej. `someKey = 'a'` , entonces `{[someKey]: 4}` es el objeto `{a: 4}` .
- ver qué devuelve `Object.assign({}, ...[{a:5},{b:8}])`


## De object literals a clases

Empecemos viendo qué pasa si el valor de un atributo es una _función_.
``` javascript
let windowSpec = {
    height: 200,
    width: 150,
    area: function() { return this.height * this.width }
} 
```

A veeeer
``` javascript
> windowSpec.area
[Function: area]
> windowSpec.area()
30000
```

Puedo **ejecutar** la función, y obtener el valor de los atributos con `this.<attrName>`.

------
**Comentario**{: style="color: SteelBlue"}:  
Esto pasa con _cualquier_ referencia a función, p.ej.
``` javascript
> const double = function(n) { return n * 2 }
undefined
> double
[Function: double]
> double(4)
8
```

------

Demos un paso más: definamos una función _que devuelva_ un objeto con atributos "mixtos" (algunos funciones, otros no).
``` javascript
function WindowSpecFn(h,w) {
  return {
    height: h,
    width: w,
    area: function() { return this.height * this.width }
  }
}
```

ya tenemos casi una clase
``` javascript
> let spec1 = WindowSpecFn(50,20)
undefined
> spec1
{ height: 50, width: 20, area: [Function: area] }
> spec1.height
50
> spec1.area()
1000
```

En rigor, la definición de clases en JavaScript es un syntax sugar de algo parecido a la definición de `WindowSpecFn`.  
Antes de ES6 que agregó la sintaxis de `class`, ya se podían definir clases. A mí me salió esto, que funciona ...
``` javascript
function WindowSpec(h,w) {
  this.height = h
  this.width = w
}

WindowSpec.prototype.area = function() { 
  return this.height * this.width 
}
```
... sin que yo termine de entender qué es eso del "prototipo". Me inspiré en [esta página de doc](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new).



## Clases - constructor y atributos

Definamos una clase `WindowSpec` con un par de agregados.

``` javascript
const moment = require('moment')
const { windowManager } = require('./ourWindowLibrary.js')

class WindowSpec {
  constructor(_height, _width) {
    this.height = _height
    this.width = _width
  }

  area() { return this.height * this.width }

  open() {
    // registro la primera vez que se abre esta ventana
    if (!this.firstOpenTime) {
      this.firstOpenTime = moment()
    }
    // la registro en el windowManager
    windowManager.manageSpec(this)
    // ... acciones para abrir la ventana ...
  }
}
```

(entre paréntesis, `moment` es **la** librería para manejar fechas, períodos, horas, etc. en el mundo JS/TS, volveremos sobre ella)

Pregunta ¿cuántos _atributos_ define esta clase?  
Para obtener la respuesta, hay que barrer **todo** el código. Para evitar esto, puede ser una buena idea definirlos todos en el constructor.  
**Nota**: este problema no existe en TS, sólo en JS. TS obliga a definir todos los atributos, esto lo retomamos al hablar de clases en TS.

Pasemos a otro asunto.  
En este ejemplo, para que una ventana se pueda usar, hay que registrarla en el `windowManager`.  
Hicimos esta implementación del manager, para testear:
``` javascript
const windowManager = {
  managedSpecs: [],
  manageSpec(spec) {
      this.managedSpecs.push(spec)
  }
}
```
simplemente mantiene una lista con las ventanas que maneja.

Si creo una ventana y la abro tres veces
``` javascript
> const spec1 = new WindowSpec(300,120)
> spec1.open()
> spec1.open()
> spec1.open()
```
¿cuántos elementos tendrá `windowManager.managedSpecs`? **Tres** ...
``` javascript
> windowManager.managedSpecs
[
  WindowSpec {
    height: 300,
    width: 120,
    firstOpenTime: Moment<2020-05-09T18:28:20+00:00>
  },
  WindowSpec {
    height: 300,
    width: 120,
    firstOpenTime: Moment<2020-05-09T18:28:20+00:00>
  },
  WindowSpec {
    height: 300,
    width: 120,
    firstOpenTime: Moment<2020-05-09T18:28:20+00:00>
  }
]
```
... que son tres referencias a la misma `WindowSpec`. Esto pasa porque el enganche el registro de la `WindowSpec` en el `windowManager` se hace en el `open`. Si movemos el registro al constructor

``` javascript
class WindowSpec {
  constructor(_height, _width) {
    this.height = _height
    this.width = _width
    // la registro en el windowManager
    windowManager.manageSpec(this)
  }

  area() { return this.height * this.width }

  open() {
    // registro la primera vez que se abre esta ventana
    if (!this.firstOpenTime) {
      this.firstOpenTime = moment()
    }
    // ... acciones para abrir la ventana ...
  }
}
```
entonces se va a registrar una vez sola, cuando se cree. Si el registro es una operación costosa, ganamos en eficiencia.

Esta idea puede servir para servicios o controllers, los registros que se tienen que hacer una sola vez, van en el constructor y no en los métodos operativos.

