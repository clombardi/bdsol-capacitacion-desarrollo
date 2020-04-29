## Acerca de TypeScript
TypeScript es un compiler (¿o transpiler?), que _genera código JavaScript_.  
El `tsc` es el compilador de TypeScript a JavaScript. En este sentido, usar `tsc` es análogo a babelizar.

¿Por qué usamos TS en lugar de JS derecho viejo?  
Lo más destacado que le agrega es un _sistema de tipos_ (de ahí la T de TypeScript), que permite hacer _chequeos estáticos_. Esto es, chequeos sobre el código antes que se ejecute. TS se puede ver como un mega-linter ...  
... pero **OJO** que también agrega otras cosas, en particular los _decorators_ que NestJS usa un montón, y de los que vamos a hablar.

En lo personal, creo que en rigor lo más valioso que brinda es la mejora al _intellisense_ de los editores.


### Atrás está JS
Es importante entender que el código que se ejecuta es JavaScript. Si p.ej. ejecutamos código generado desde TypeScript en Node, vemos que
- lo que se carga en Node es el `.js` que genera `tsc`, si cargamos directamente un `.ts` da error.
- en la consola de Node no podemos usar tipos.

Cuando probamos "desde NestJS", es Nest quien se encarga de la compilación. Pero siempre está.

### Opciones de compilador
Por lo general, `tsc` toma las opciones de compilación de un archivo llamado `tsconfig.json`. El VS Code también tiene en cuenta ese archivo para mostrar errores.

