---
layout: default
---

# TypeORM - opciones de búsqueda
Los repositorios de TypeORM incluyen varias operaciones para obtener entidades, la "R" de "CRUD". En las páginas anteriores, aparecieron `find` y `findOneOrFail`.

En el ejemplo de `find` incluido en la descripción de cómo describir operaciones, que tiene la forma
```typescript
await applicationRepository.find();
```
la respuesta incluye **todas** las solicitudes de cuenta que estén registradas en la BD.

Por su parte, el objetivo de `findOneOrFail` es obtener _una sola entidad_. Mediante la opción `where`, se especifica cuál es la entidad que queremos obtener en base a ciertas _condiciones_. En los ejemplos que aparecieron en la página sobre relaciones, como
```typescript
await applicationRepository.findOneOrFail({ 
    where: { customer: 'Felicitas Guerrero' }, /* otras opciones */
});
```
la condición se refiere al _valor exacto_ de un atributo. En este caso, se está buscando por `Felicitas Guerrero`, eventuales solicitudes cuyo cliente sea p.ej. `Felicitas Guerreroyola`, `Felicitas Guerre` o `Felicita Guerrero` no serían resultados posibles de esta operación.

Las operaciones de `find...` de TypeORM aceptan muchas variantes para especificar condiciones de búsqueda, que en rigor expresan las distintas condiciones que pueden aparecer y combinarse en la cláusula `WHERE` de un `SELECT` en SQL.  
Podemos pensar que los `find...` nos permiten expresar `SELECT` de SQL, con una notación simplificada para los `JOIN` que es la opción `relations`, y un modelo de las condiciones del `WHERE` como objetos que admiten una manipulación más sencilla que un string largo.

La [página de la doc sobre opciones de búsqueda](https://typeorm.io/#/find-options) describe las opciones disponibles. En lo que sigue, vamos a describir algunas a modo de síntesis.  
Veremos que en algunos casos, en particular cuando se quiere expresar condiciones sobre entidades relacionadas, las operaciones `find...` nos "quedan cortas". 
En tales casos, conviene recurrir a otro mecanismo que provee TypeORM para describir búsquedas, los `QueryBuilder`, que permiten una sintaxis aún más parecida a la de un `SELECT` SQL, mucho más potentes en las consultas que se pueden expresar, y al mismo tiempo más complicados.


## Una aclaración antes de arrancar
En todos los ejemplos que siguen, vamos a especificar las condiciones de búsqueda como el valor del atributo `where` en las opciones del `find...`, o sea, los ejemplos van a tener esta forma 
```typescript
await applicationRepository.find({ where: { /* condiciones */ } });
```
En la mayor parte de los casos, la única opción que vamos a incluir es `where`. En estos casos, o sea si al hacer un `find` queremos especificar _únicamente_ condiciones de búsqueda, podemos especificar directamente las condiciones de búsqueda como opciones del `find`, "ahorrándonos" el `where`. O sea, en lugar de lo anterior podríamos escribir
```typescript
await applicationRepository.find({ /* condiciones */ });
```

En esta página preferimos incluir el `where` para que sea sencillo copiar estos ejemplos y agregarle más condiciones.


## Valor exacto
Como ya apareció en ejemplos anteriores, para indicar una búsqueda por valor exacto de un atributo, se especifica un atributo `{ nombre: valor }`. P.ej. para obtener las solicitudes en estado pendiente, podemos indicarlo de esta forma.
```typescript
await applicationRepository.find({ where: { status: Status.PENDING } });
```


## Búsquedas con un solo resultado, búsquedas por clave primaria
Suponiendo que sólo puede haber una solicitud para cada cliente, queremos obtener el status de la solicitud de cuenta (única) de Ana Bolena. Escribimos lo siguiente.
```typescript
const application = await applicationRepository.find({ where: { customer: 'Ana Bolena' } });
const statusDeBolena = application.status
```
... y no compila. ¿Qué pasó? Que el resultado del `find` es _una lista_. 

TypeORM incluye variantes del `find` orientadas a búsquedas con un solo resultado. Son `findOne` y `findOneOrFail`. Usando cualquiera de las dos podemos resolver la obtención del status de la solicitud de Ana Bolena 
```typescript
const application = await applicationRepository.findOne({ where: { customer: 'Ana Bolena' } });
const statusDeBolena = application.status
```

La diferencia entre `findOne` y `findOneOrFail` es el comportamiento si no se encuentra ninguna entidad que cumpla con los criterios especificados. En tal caso, `findOne` devuelve `undefined`, mientras que `findOneOrFail` lanza un error de tipo `EntityNotFound`.

Para búsquedas por valor de la clave primaria (en nuestros ejemplos, el `id`), las operaciones `findOne` y `findOneOrFail` ofrecen una notación simplificada, en la que se le pasa directamente el valor. O sea, las siguientes tres expresiones son equivalentes.
```typescript
await applicationRepository.findOne(4);
await applicationRepository.findOne({ id: 4 });
await applicationRepository.findOne({ where: { id: 4 } });
```


## Otras condiciones: comparaciones, conjunto de posibles valores
Todos los casos descriptos hasta ahora se refieren al _valor exacto_ de un atributo. TypeORM define operadores que permiten especificar otras condiciones. Estos operadores se incluyen dentro de la especificación del `while` para un atributo.

Por ejemplo, podemos obtener las solicitudes que requieren de 4 o más aprobaciones, podemos mediante esta expresión.
```typescript
await applicationRepository.find({ where: { requiredApprovals: MoreThanOrEqual(4) } }});
``` 
Así como está `MoreThanOrEqual`, tenemos también `MoreThan`, `LessThanOrEqual`, `LessThan`, y otras condiciones que se detallan [en la doc](https://typeorm.io/#/find-options/advanced-options).

Destacamos el operador `In`, para describir un conjunto de posibles valores. P.ej. para obtener las solicitudes que requieran o bien 4 o bien 7 aprobaciones, podemos hacerlo mediante esta expresión.
```typescript
await applicationRepository.find({ where: { requiredApprovals: In([4,7]) } });
```


## Operadores lógicos: not / and / or
Otro de los operadores que incluye TypeORM es `Not`, que "invierte" una condición. Si se aplica directamente sobre un valor, entonces expresa la condición de "distinto". P.ej. esta expresión
```typescript
await applicationRepository.find({ where: { status: Not(Status.PENDING) } }});
```
permite obtener las solicitudes cuyo status sea distinto de `Status.PENDING`.

También se puede aplicar el operador `Not` a otro operador, para negar la condición correspondiente. P.ej. la expresión
```typescript
await applicationRepository.find({ where: { 
    status: Not(In([Status.PENDING, Status.ANALYSING])) 
} });
```
describe la búsqueda de solicitudes cuyo status no sea ni `PENDING` ni `ANALYSING`.

Para incluir distintas condiciones de forma tal que todas deben cumplirse (o sea, un "and" de condiciones), sencillamente se colocan todas como distintos atributos del objeto que representa el `where`. P.ej, si queremos obtener las solicitudes en estado `PENDING` o `ANALYSING`, que requieran de cuatro o más aprobaciones, podemos expresarlo mediante esta expresión.
```typescript
await applicationRepository.find({ where: { 
    status: In([Status.PENDING, Status.ANALYSING]), 
    requiredApprovals: MoreThanOrEqual(4)
} });
```

Para expresar un "or" de condiciones, o sea, indicar varias condiciones con la intención de obtener las entidades que cumplan _al menos una_ de ellas, debemos transformar el objeto asociado al `where` en una lista, donde cada elemento representa una de las posibles condiciones.  
P.ej., para obtener las solicitudes para las cuales, _o bien_ su estado es `PENDING` o `ANALYSING`, _o bien_ requieren de cuatro o más aprobaciones, podemos construir esta expresión.
```typescript
await applicationRepository.find({ where: 
    [
        { status: In([Status.PENDING, Status.ANALYSING]) }, 
        { requiredApprovals: MoreThanOrEqual(4) }
    ]
});
```


## Like
Otra característica de las búsquedas de SQL que se refleja en TypeORM es la cláusula `LIKE`, en la que se puede indicar la forma esperada de una columna de tipo string. TypeORM define el operador `LIKE`, similar a los de comparaciones o conjuntos posibles de valores.  
P.ej. para obtener las solicitudes para las cuales el apellido del cliente termina en `Bolena`, podemos hacerlo mediante esta expresión.
```typescript
await applicationRepository.find({ where: { customer: Like('%Bolena') } });
```

## IsNull
Con el operador `IsNull` podemos obtener las entidades que no tengan un determinado dato cargado. P.ej. podemos obtener las sucursales para las que no se cargó el dato de superficie de esta forma
```typescript
await agencyRepository.find({ where: { area: IsNull() } });
```


## Condiciones sobre entidades relacionadas - QueryBuilder
Hasta ahora, todas las condiciones están asociadas a atributos de la misma entidad sobre la que se hace find. Veamos ahora cómo hacer algunas consultas relativas a _entidades relacionadas_. En estos ejemplos, vamos a usar la relación entre solicitudes y análisis crediticios, mencionada en la página sobre relaciones. 

Utilizando el operador `IsNull`, se pueden obtener las solicitudes que _no tengan asociado_ un análisis crediticio.
```typescript
await applicationRepository.find({ where: { creditAssessment: IsNull() } });
```

Para expresar condiciones sobre _atributos_ de entidades relacionadas, no lo vamos a poder hacer usando los métodos `find` y análogos. P.ej. supongamos que queremos obtener las solicitudes que tengan un análisis crediticio asociado, cuyo límite de crédito asignado sea 240000 pesos. Lo que sigue son dos posibles formas en la que, en princpio, podría expresarse esta condición en el `where` de un `find`.
```typescript
await applicationRepository.find({ where: { creditAssessment.creditLimit: 240000 } });
```
o
```typescript
await applicationRepository.find({ where: { creditAssessment: {creditLimit: 240000} } });
```
De estas dos opciones, la primera da error en el editor o compilador, la segunda toma `{creditLimit: 240000}` como el valor literal del atributo `creditAssessment`. O sea, ninguna de las dos variantes tiene el resultado esperado.

### Query builders
Para expresar este tipo de condiciones, tenemos que utilizar otra de las herramientas que brinda TypeORM para expresar consultas, los llamados `QueryBuilder`s.  
Un `QueryBuilder` es un objeto que sirve para especificar una consulta de un modo similar a SQL, pero donde las distintas cláusulas se expresan mediante objetos JS/TS.  
En lo que sigue vamos a describir _muy brevemente_ algunas características de los `QueryBuilder`. Se puede encontrar una descripción completa [en la doc de TypeORM](https://typeorm.io/#/select-query-builder). Ahí se puede ver que también se pueden generar `QueryBuilder`s para operaciones de alta, baja o modificación.

Para crear un `QueryBuilder`, podemos usar la operación `createQueryBuilder` que aceptan los repositorios. Para nuestro ejemplo.
```typescript
const query = await applicationRepository.createQueryBuilder('aa');
```
Esto crea un query cuya cláusula `FROM` se va a referir a la tabla donde se persisten las solicitudes de cuenta. El parámetro indica el alias que va a tener la tabla dentro del query. En este caso, la query va a generar una consulta SQL de este estilo
```sql
SELECT *
FROM account_applications aa
```
Para agregar la entidad relacionada (en este caso los análisis crediticios) a la consulta, se puede hacer usando la operación `leftJoinAndSelect`, indicando el atributo correspondiente a la entidad relacionada, y el alias que va a tener la tabla de esa entidad en el query. 
```typescript
query.leftJoinAndSelect('aa.creditAssessment', 'ca');
```
Después de hacer el join, se pueden expresar condiciones sobre la entidad relacionada. Para describir las condiciones se utiliza la operación `where`.
```typescript
query.where('ca.creditLimit = 240000');
```

Esta es la definición completa del QueryBuilder, seguida por el pedido de que ejecute la búsqueda mediante la operación `getMany()`.
```typescript
const query = await applicationRepository.createQueryBuilder('aa');
query.leftJoinAndSelect('aa.creditAssessment', 'ca');
query.where('ca.creditLimit = 240000');
await query.getMany();
```
El SQL que genera y ejecuta tiene esta forma (aproximada)
```sql
SELECT *
FROM account_applications aa
LEFT JOIN credit_assessments ca on aa.creditAssessmentId = ca.id
WHERE ca.creditLimit = 240000
```

El resultado es una lista de objetos `AccountApplication`, que van a incluir el `CreditAssessment` de cada una, o sea un resultado análogo a hacer un 
```typescript
applicationRepository.find({ where: /* condicion */, relations: ['creditAssessment']}); 
```

Este es un ejemplo de posible resultado, en el que hay una sola solicitud que cumple la condición.
```typescript
[
  AccountApplication {
    id: 6,
    customer: 'Mercedes Ponce',
    status: 'Pending',
    date: null,
    requiredApprovals: 2,
    creditAssessment: CreditAssessment {
      id: 4,
      customer: 'Mercedes Ponce',
      creditLimit: '240000.00'
    }
  }
]
```

### Variantes en los QueryBuilders
Veamos algunas de las (muchísimas) variantes que se pueden utilizar al armar un QueryBuilder.

**Parámetros en el `where`**  
Los valores de los parámetros pueden separarse del string de la cláusula `where`.
```typescript
query.where('ca.creditLimit = :creditLimit', { creditLimit: 240000 });
```
Este formato del `where` es útil cuando estos valores vienen p.ej. en un request. 

**No incluir los objetos relacionados**  
La operación `leftJoinAndSelect` hace que la tabla asociada a una entidad se incorpore a la query mediante un `LEFT JOIN`, y además, que esta entidad se incluya en el resultado. Esto es lo que se especifica mediante "andSelect".  
Si se usa `leftJoin` en lugar de `leftJoinAndSelect`, se incorpora el `LEFT JOIN`, pero la entidad no se incluye en el resultado. Esto sirve p.ej. si queremos incluir condiciones en el `where` pero no nos interesa incluir la entidad relacionada en el resultado.  
Si en el ejemplo anterior cambiamos `leftJoinAndSelect` por `leftJoin`, este es el resultado que se obtiene.
```typescript
[
  AccountApplication {
    id: 6,
    customer: 'Mercedes Ponce',
    status: 'Pending',
    date: null,
    requiredApprovals: 2
  }
]
```

**Variantes al `getMany`**  
Las QueryBuilder incluyen varias operaciones para obtener resultados. Entre ellas mencionamos: 
- `getOne` y `getOneOrFail`, que son análogas a `findOne` y `findOneOrFail`, para obtener un solo resultado.
- `getSql` que sirve para observar cuál es el SQL generado por la query.
- `getCount` y `getManyAndCount`, que devuelven sólo la cantidad de resultados, o bien agregan la cantidad de resultados.
- `getRawMany` y `getRawOne`, que devuelven objetos planos, que no se corresponden a las entidades mapeadas. 

**Operaciones de agregación**  
Mediante QueryBuilder también se pueden realizar consultas agregadas, en los que se obtienen valores como `SUM`, `MAX`, `AVG`, etc.. Estas funciones se especifican mediante la operación `select`. En el siguiente ejemplo, se consulta el total de `creditLimit` para los análisis de crédito vinculados a solicitudes en estado `Pending`.
```typescript
const query = await applicationRepository.createQueryBuilder('aa');
query.select("SUM(ca.creditLimit)", "totalCreditLimit");
query.innerJoin('aa.creditAssessment', 'ca');
query.where("aa.status = 'Pending'")
await query.getRawOne();
```
Obsérvese el uso de `getRawOne`, dado que el resultado de esta query no puede modelarse como una de las entidades mapeadas. En este ejemplo, el resultado tiene la siguiente forma
```typescript
{ totalCreditLimit: '325000.00' }
```
También se pueden incluir cláusulas `groupBy`, en tal caso se utiliza `getRawMany` para obtener los resultados.


### Subqueries
Vayamos a otro caso: queremos obtener las solicitudes que tengan el mismo status que la solicitud de Ana Bolena. 
En SQL podemos resolver esta consulta mediante una única query, utilizando una **subquery** para expresar la condición sobre el status.
```typescript
SELECT *
FROM account_applications
WHERE status = (SELECT status FROM account_applications WHERE customer = 'Ana Bolena');
```
Los QueryBuilders permiten expresar queries que incluyen subqueries. 
Una de las varias formas de definir este tipo de consultas consiste en definir la subquery por separado, y usarla en la query principal pidiéndole el `getQuery()`. A su vez, hay que agregar los eventuales parámetros de la subquery en la query principal.  
La expresión usando QueryBuilders para describir la consulta mencionada recién queda así.
```typescript
const queryBolena = await applicationRepository.createQueryBuilder('aa2');
queryBolena.select("aa2.status");
queryBolena.where("aa2.customer = :customer", { customer: 'Ana Bolena'});

const queryStatusComoBolena = await applicationRepository.createQueryBuilder('aa');
queryStatusComoBolena.where(`aa.status = (${queryBolena.getQuery()})`);
queryStatusComoBolena.setParameters(queryBolena.getParameters());
```
Hay que definir el `select` de la subquery, para que incluya únicamente el atributo `status`.


## Otras características
Mencionamos dos herramientas adicionales que pueden utilizarse en las búsquedas, los detalles pueden verse en la documentación de TypeORM.

1. Entre las opciones del `find` encontramos `skip` y `take`, que sirven para manejar datos en forma paginada. Los QueryBuilder incluyen las operaciones `skip(n)` y `take(n)`, que tienen el mismo comportamiento.

2. Entre los operadores de búsqueda (o sea, al nivel de `MoreThan`, `IsNull` o `Not`), encontramos uno llamado `Raw`, que permite introducir SQL dentro de las búsquedas con `find`, sin tener que pasar a QueryBuilder. No sirve para manejar datos de entidades relacionadas, pero sí para algunas condiciones especiales en búsquedas que involucran un solo tipo de entidad.



## Para practicar 
Obtener las solicitudes que tengan asociado un `CreditAssessment`. ¿Aparece el `CreditAssessment` asociado a cada una?

Obtener las solicitudes cuya cantidad de aprobaciones requeridas no sea ni 4 ni 7, y cuyo status sea o bien `Status.PENDING` o bien `Status.ANALYSING`.

Obtener las solicitudes de la sucursal cuyo código es `'022'`.

Obtener las sucursales que tengan el texto `San Martín`, o bien en la dirección, o bien en el nombre de la sucursal.

Obtener las solicitudes de clientes cuyo apellido es, o bien `Bolena`, o bien `Molina`. Notar las diferencias entre el `LIKE` de SQL y las expresiones regulares que permite Mongo.

Obtener las sucursales que no tengan cargada ciudad.

Obtener las solicitudes de (sucursales que estén en la provincia de) Córdoba. Sólo los datos de cada solicitud, no incluir datos de sucursales ni de ciudades.  
**Hint** no son necesarias subqueries.

Obtener el promedio de límite de crédito de los análisis crediticios.

Obtener las solicitudes que tienen análisis crediticios cuyo límite de crédito es superior al promedio. Considerar el uso de subqueries.

Obtener la cantidad de solicitudes por sucursal, usando `groupBy`.
