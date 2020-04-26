## Valores falsy y truthy

**Todos** los valores en JavaScript tienen una "correspondencia" booleana, o sea que se consideran (de alguna forma) análogos a `true` o a `false`. A los valores análogos a `true` se los llama _truthy_, y _falsy_ a los que son análogos a `false`.

De esta forma, los operadores lógicos `&&`, `||`, `!`, pueden aplicarse a **cualquier** valor. P.ej. la expresión

```
"hola" || "amigos"
```

es válida, y su resultado es ... `"hola"`. Analicemos cómo se llega a este resultado

1. los operandos de `||` y `&&` se evalúan de izquierda a derecha.
1. `"hola"` es un valor _truthy_.
1. si en una disyunción uno de los operandos es verdadero, la disyunción es verdadera sin importar el valor del otro operando. <br/> En sencillo: "true `or` lo_que_sea da true".
1. JavaScript se aprovecha de esto: si el operando izquierdo de un `||` es _truthy_, la evaluacion termina _sin mirar el operando derecho_. <br/> O sea, JavaScript aplica _short-circuit evaluation_, ver en [la documentación MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_Operators) (buscar "short-circuit evaluation"), o la sección 12.13.3 en [la spec ECMAScript](https://www.ecma-international.org/ecma-262/10.0/index.html).

Si el valor del operando izquierdo es _falsy_, entonces se devuelve el valor del operando derecho. P.ej. el resultado de 

```
null || "amigos"
```
es `"amigos"`, porque `null` es _falsy_, y por lo tanto, el resultado del `||` es el operando derecho.


Con la conjunción se da el mismo efecto:

| operación | resultado | aclaración o pregunta |
| --- | --- | --- |
| `null && "amigos"` | `null` | porque "false `and` lo_que_sea da false" | 
| `"hola" && "amigos"` | `"amigos"` | ¿por qué `"amigos"` y no `"hola"`? |

<br/>

------
**Muy importante**{: style="color: SteelBlue"}:  
notar que el resultado de `"hola" || "amigos"` **no** es `true`, sino `"hola"`.

------
Esto habilita varios trucos, en los que el uso de `||` y `&&` permite acortar el código.






