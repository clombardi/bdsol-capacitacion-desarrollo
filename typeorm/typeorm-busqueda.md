---
layout: default
---

# TypeORM - opciones de búsqueda
Los repositorios de TypeORM incluyen varias operaciones para obtener entidades, la "R" de "CRUD". En las páginas anteriores, aparecieron `find` y `findOneOrFail`.

En el ejemplo de `find` incluido en la descripción de cómo describir operaciones, que tiene la forma
```typeorm
await applicationRepository.find();
```
la respuesta incluye **todas** las solicitudes de cuenta que estén registradas en la BD.

Por su parte, el objetivo de `findOneOrFail` es obtener _una sola entidad_. Mediante la opción `where`, se especifica cuál es la entidad que queremos obtener en base a ciertas _condiciones_. En los ejemplos que aparecieron en la página sobre relaciones, como
```typeorm
await applicationRepository.findOneOrFail({ 
    where: { customer: 'Felicitas Guerrero' }, /* otras opciones */
});
```
la condición se refiere al _valor exacto_ de un atributo. En este caso, se está buscando por `Felicitas Guerrero`, eventuales solicitudes cuyo cliente sea p.ej. `Felicitas Guerreroyola`, `Felicitas Guerre` o `Felicita Guerrero` no serían resultados posibles de esta operación.

Las operaciones de `find...` de TypeORM aceptan muchas variantes para especificar condiciones de búsqueda, que en rigor expresan las distintas condiciones que pueden aparecer y combinarse en la cláusula `WHERE` de un `SELECT` en SQL.  
Podemos pensar que los `find...` nos permiten expresar `SELECT` de SQL, con una notación simplificada para los `JOIN` que es la opción `relations`, y un modelo de las condiciones del `WHERE` como objetos que admiten una manipulación más sencilla que un string largo.

La [página de la doc sobre opciones de búsqueda](https://typeorm.io/#/find-options) describe las opciones disponibles. En lo que sigue, vamos a describir algunas a modo de síntesis.  
Veremos que en algunos casos, en particular cuando se quiere expresar condiciones sobre entidades relacionadas, las operaciones `find...` nos "quedan cortas". 
En tales casos, conviene recurrir a otro mecanismo que provee TypeORM para describir búsquedas, los `QueryBuilder`, que permiten una sintaxis aún más parecida a la de un `SELECT` SQL.


## Una aclaración antes de arrancar
En todos los ejemplos que siguen, vamos a especificar las condiciones de búsqueda como el valor del atributo `where` en las opciones del `find...`, o sea, los ejemplos van a tener esta forma 
```typeorm
await applicationRepository.find({ where: { /* condiciones */ } });
```
En la mayor parte de los casos, la única opción que vamos a incluir es `where`. En estos casos, o sea si al hacer un `find` queremos especificar _únicamente_ condiciones de búsqueda, podemos especificar directamente las condiciones de búsqueda como opciones del `find`, "ahorrándonos" el `where`. O sea, en lugar de lo anterior podríamos escribir
```typeorm
await applicationRepository.find({ /* condiciones */ });
```

En esta página preferimos incluir el `where` para que sea sencillo copiar estos ejemplos y agregarle más condiciones.


## Valor exacto
Como ya apareció en ejemplos anteriores, para indicar una búsqueda por valor exacto de un atributo, se especifica un atributo `{ nombre: valor }`. P.ej. para obtener las solicitudes en estado pendiente, podemos indicarlo de esta forma.
```typeorm
await applicationRepository.find({ where: { status: Status.PENDING } });
```


## Búsquedas con un solo resultado, búsquedas por id
Suponiendo que sólo puede haber una solicitud para cada cliente, queremos obtener el status de la solicitud de cuenta (única) de Ana Bolena. Escribimos lo siguiente.
```typeorm
const application = await applicationRepository.find({ where: { customer: 'Ana Bolena' } });
```


