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

(pregunta: ¿qué pasa si hago `windowSpec = 100`, también cambian todos?)

Distinta es la cosa si hacemos
``` javascript
let otherSpec = {...windowSpec}
```
porque se está generando un _clon_ de `windowSpec`. Parece que los "tres-puntos" [tienen una variedad de usos](https://dev.to/blacksonic/the-tale-of-three-dots-in-javascript-4287). Este es un syntax sugar para `Object.assign()`, o sea, un _shallow copy_. Ver la [diferencia entre shallow copy y deep copy](https://levelup.gitconnected.com/difference-between-shallow-and-deep-copy-c0a968e89c44).

Los "tres-puntos" permiten mergear varios objetos, y también agregar/modificar valores.

``` javascript
let point = {x:8, y:12, z:-4}
let otherSpec = {...windowSpec, ...point, width:75, borderWidth:4}
```

Un efecto distinto a los dos anteriores se logra mediante
``` javascript
let otherSpec = { windowSpec }
```
que es simplemente una abreviatura para `{ windowSpec: windowSpec }`.

------
**Comentario**{: style="color: SteelBlue"}:  
Lo de las referencias compartidas y los "tres-puntos" corre también para _arrays_.  
De hecho ... **los arrays son objetos**, probar `Object.keys(['a', 'b', 'c'])`, notar la similitud entre `windowSpec['width']` y `someArray[1]`.

------

### Desafíos

Armar un objeto `x` tal que si defino `y = {...x}` y hago algún cambio "dentro" de `x` (o sea, hago `x.<cosas> = <nuevoValor>`), se modifica también algo en `y`.  

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

Definir una función que, dado un objeto cuyos valores son todos numéricos, devuelve un objeto con las mismas keys, y cada value el doble del value original. P.ej.
``` javascript
> doubledObject({x:4,y:0,z:1,w:9,f:0})
{x:8,y:0,z:2,w:18,f:0}
```
Otra vez, más desafío si sale con una expresión.
A mí me salió usando `Object.values` más estas dos cosas:

- si tengo p.ej. `someKey = 'a'` , entonces `{[someKey]: 4}` es el objeto `{a: 4}` .
- ver qué devuelve `Object.assign({}, ...[{a:5},{b:8}])`







