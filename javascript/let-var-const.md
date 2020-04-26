## `let` y `var` - parecidos pero (un poquito) distintos

Ya sabemos que sirven para lo mismo, veamos las diferencias, que son dos.

### Distintos alcances
La (que yo creo) principal, tiene que ver con el _alcance_, en particular cuando se definen _scopes_ internos dentro de funciones ... lo que es más común de lo que parece, el `if`, los distintos `for` o `foreach`, las array function con llaves, todos definen scopes internos.

Vamos a la diferencia entre `let` y `var`. El alcance de `let` es el scope en el que se define. El alcance de `var` es toda la función. 

Para verlo con un par de ejemplos: esta función 
``` javascript
function letVar1() {
  {
    let a = 1
    var b = 2
  }
  return b
}
```
devuelve `2`, pero si cambio el `return b`  por `return a`, se rompe. El alcance de `b` se limita al bloque interior.

En este caso
``` javascript
function letVar2() {
  let a = 11
  var b = 12
  {
    let a = 21
    var b = 22
  }
  return b
}
```
la función devuelve `22`; hay **una sola** `b` en toda la función. 
Si cambiamos el `return b`  por `return a`, devuelve `12`. **Hay dos** `a`, el "de afuera" y el "de adentro".

### Para jugar
Algunas preguntas
- ¿se podrá usar en bloque "más adentro" un `let` definido "más afuera"?
- ¿se puede definir dos veces _el mismo_ `var` en _el mismo_ scope? ¿Qué pasa con `let`?

### La otra diferencia - cuestión histórica de JS en los browsers
En una página HTML, los `var` definidos en un `<script>` definen atributos del objeto `window`, los `let` no.

P.ej.

``` html
<script>
    var a = 42
    let b = 5

    function showVar() {
      console.log(a)
      console.log(b)
      console.log(window.a)
      console.log(window.b)
    }
</script>
```
Muestra esto en la consola
```
42
5
42
undefined
```

No sé cuánto impacto tiene esto en los tiempos de 
React / Angular, yo lo cuento por las dudas.



### Conclusión personal

Yo uso `let` siempre (que no me olvido), se porta como indica la academia.


## Const ... no cambia ¿hasta dónde?

El `const` está claro: es _constante_, lo que defino como `const` no se puede cambiar.

Ahora ¿qué es lo _constante_ en un `const`? Veamos. Si tenemos

```
const princesa = { nombre: "Carolina", apellido: "Casiraghi" }
```

no vale hacer 
`
princesa = { nombre: "Máxima", apellido: "Orange" }
`

Pero ¿qué pasa si hago `princesa.apellido = "Orange"`? ¡
Lo toma!


------
**Moraleja**{: style="color: SteelBlue"}:  
Lo constante es la _referencia_ de `princesa` al objeto. El objeto no está sellado. Para sellar el objeto está `Object.freeze()`.

Aclaración: `Object.seal()` es una variante más débil).

------

### `readonly`: un primito en el mundo TypeScript
Silly girl