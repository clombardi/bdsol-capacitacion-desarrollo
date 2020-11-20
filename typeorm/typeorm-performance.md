---
layout: default
---

# TypeORM - algunas técnicas para mejorar la performance
En esta página vamos a repasar algunas ideas que pueden servir para mejorar la performance de las operaciones que involucran accesos a BD relacionales, y qué herramientas nos brinda TypeORM para implementarlas.

## Indices
La técnica más potente para agilizar las operaciones de lectura es la _definición de índices_, de acuerdo a los criterios de búsqueda que se necesiten en una aplicación o servicio. Los efectos del índice son apreciables para tablas grandes, en el orden de miles de registros o más.

Para evaluar el efecto de la definición de un índice en una tabla, se trabajó con este modelo de sucursales que describimos [al hablar de relaciones](./typeorm-mapeo-relaciones). Se generaron dos tablas con 15 millones de sucursales cada una, en un Postgres 11 local. A una de las tablas se le agregó un índice por `code`. Se implementó un servicio Nest con un endpoint `GET` que hace cuatro búsquedas de sucursal por código.
```typescript
/* en el controller */
@Get('rangeByCode/:initialCode')
async getAgencyRangeByCode(@Param("initialCode") initialCode: string): Promise<Agency[]> {
    return this.agenciesService.getAgencyRangeByCode(initialCode, 4);
}

/* en el provider */
async getAgencyRangeByCode(initialCode: string, count: number): Promise<Agency[]> {
    const initialCodeAsNumber = Number(initialCode);
    // la función padStart es gentileza de lodash
    const numberToCode = number => padStart(String(number), 8, '0');
    const result: Agency[] = []
    for (let index = 0; index < count; index++) {
        result.push(await this.getAgencyByCode(numberToCode(initialCodeAsNumber + index)));
    }
    return result;
}
```

Este es el resultado de acceder al endpoint apuntando a la tabla **sin** índice.
![cuatro queries sin índice](./agency-range-no-index.jpg)

Al cambiar a la tabla **con** índice, se obtiene lo siguiente.
![cuatro queries con índice](./agency-range-index.jpg)

Para endpoints de acceso frecuente, se pueden lograr ganancias importantes de performance si se definen índices que se correspondan con el criterio de búsqueda.


### Soporte para índices en TypeORM
Los índices son _recursos propios de las BD_. Los programas que acceden a la BD desconocen (al menos en la generalidad de los casos) qué índices están definidos, y no necesitan esta información para hacer consultas. El SQL de la consulta que se envía a la base es el mismo, haya o no índices que apliquen. Es la BD la que detecta la existencia de índices, y los aprovecha para dar una respuesta más rápida.

TypeORM brinda soporte para indicar qué índices deben contemplarse en la tabla que modele cada entidad, para esto define el decorator `@Index`. P.ej. en la definición de la entidad `Agency`, se puede indicar que debe crearse un índice sobre el atributo `code` de esta forma.
```typescript
@Entity({ name: 'agencies' })
export class Agency {
    @PrimaryGeneratedColumn()
    id: number

    @Column({ length: 10 })
    @Index()
    code: string

    @Column({ length: 120 })
    name: string

    @Column({ nullable: true })
    address: string

    @Column({ nullable: true })
    area: number

    @ManyToOne(() => City)
    city: City;
}
```
Esta información se utiliza **únicamente** en la sincronización que hace TypeORM, cuando levanta la aplicación, entre las definiciones de entidades, y el esquema de la BD a la que se está conectando. Si la entidad incluye la definición de índices que no están en la BD, entonces TypeORM le indica a la BD que cree índices que cumplan con la definición.

Este soporte incluye varias variantes (p.ej. índices que involucran varias columnas), que se pueden consultar [en la página sobre índices de la doc de TypeORM](https://typeorm.io/#/indices).

Los índices también pueden ser creados en la misma BD, mediante la sentencia `CREATE INDEX`. En Postgresql, esta expresión crea un índice sobre la columna `code`.
```sql
CREATE INDEX ix_code ON agencies (code);
```
Pero **atención**, si se levanta una aplicación que usa TypeORM, que no tiene definido el índice en el mapeo de entidad, y que tiene `synchronize=true` en las opciones de conexión, TypeORM va a _borrar_ el índice al levantarse la app (o al menos eso es lo que indica la doc de TypeORM).
Para evitar esto, se pueden declarar "índices a respetar" en la definición de la entidad. Para el ejemplo del índice `ix_code`, queda así.
```typescript
@Entity({ name: 'agencies' })
@Index("ix_code", { synchronize: false })
export class Agency {
    /* ... properties ... */
}
```


### Otros comentarios sobre índices
**Distintos tipos de índices**  
Al trabajar con [índices en MongoDB](../mongoose-performance/indices), mencionamos la existencia de distintos tipos de índices. En el caso de las BD relacionales, los tipos de índice que se pueden definir van a depender fuertemente del motor de BD que se utilice (MySQL/MariaDB, Postgresql, Oracle, SQLServer, etc.).  
TypeORM puede manejar sólo índices "clásicos" (como el que describimos sobre el código de las sucursales) y espaciales. Para utilizar índices con características que TypeORM no maneja, deberán definirse estos índices sobre la BD, e indicar en el mapeo de entidad de TypeORM que "respete" los índices definidos.

**Frecuencia de lectura vs frecuencia de escritura**  
La definición de índices no es gratuita. Los índices son recursos adicionales que debe manejar el motor de BD. En particular, para cada operación que involucre modificaciones en la BD (altas, bajas o modificaciones de entidades), la BD debe actualizar los índices de las tablas modificadas. 
En consecuencia, al mismo tiempo que los índices brindan ventajas que pueden ser considerables respecto de las _operaciones de lectura_, afectan negativamente la performance de las _operaciones de escritura_.

Es por esto que para decidir si se va a crear o no un índice en una tabla, conviene comparar la frecuencia de operaciones de lectura que se ven beneficiadas, con la de operaciones de escritura que se ven perjudicadas.  
En una tabla que se escribe mucho y se lee poco (como podría ser el caso de tablas de log), agregar muchos índices puede generar más perjuicio que beneficio. En el otro extremo, en tablas con mucha lectura y poca escritura (como podría ser el padrón electoral en la aplicación de consulta de padrones), agregar índices es particularmente conveniente.

**Herramientas para generar índices**  
Para cerrar lo que vamos a decir sobre índices, mencionamos que existen herramientas que recolectan estadísticas sobre el uso de una BD, y a partir de esta información recomiendan qué índices conviene crear para mejorar la performance.  
Este redactor sólo sabe que tales herramientas existen, nunca experimentó con tales artefactos. Por eso se limita a mencionar su existencia.


## Operaciones masivas
La sintaxis de SQL incluye la posibilidad de que una única operación afecte a muchas filas. Veamos algunos casos, y qué herramientas nos da TypeORM para manejarlos.


### Actualizaciones masivas mediante un único UPDATE
Volviendo a las solicitudes de cuenta que describimos [al presentar los conceptos básicos de TypeORM](./typeorm/typeorm-bases), podemos p.ej. modificar el valor de la propiedad `requiredApprovals` para todas las solicitudes que estén en estado `Pending`, con esta única sentencia
```sql
UPDATE account_applications 
SET "requiredApprovals" = "requiredApprovals" + 1 
WHERE status = 'Pending'
```
Esta sentencia es mucho más eficiente que obtener primero la lista de solicitudes en estado `Pending`, y luego aplicar un `UPDATE` a cada una, identificándolas p.ej. por el `id`.

TypeORM permite expresar estas operaciones masivas, mediante QueryBuilders. En el caso particular del `UPDATE` recién mencionado, podemos definirlo como se ve en este método de servicio.
```typescript
async makePendingApplicationsMoreDemanding(): Promise<MassiveOperationDTO> {
    const massiveUpdateResult = await this.applicationRepository
        .createQueryBuilder()
        .update()
        .set({ requiredApprovals: () => '"requiredApprovals" + 1' })
        .where(`status = '${Status.PENDING}'`)
        .execute();
    return { count: massiveUpdateResult.affected }
}
```
Notamos que al QueryBuilder se le indica que debe realizar un `update()`, y a continuación las cláusulas `set(...)` y `where(...)`. En el `set`, se pasa un objeto indicando los atributos a modificar, y el valor a asignarle a cada uno. Cuando el nuevo valor depende del estado actual de la entidad, se indica mediante una función como se ve en el ejemplo.  
El resultado incluye al atributo `affected`, que indica la cantidad de filas modificadas.  
Los QueryBuilder para armar sentencias `UPDATE` puede incorporar otros elementos mencionados [en la página sobre búsqueda](./typeorm-busqueda), p.ej. subqueries.

Estos usos de QueryBuilder se describen en páginas de la doc de TypeORM, que se refieren al [update](https://typeorm.io/#/update-query-builder), [insert](https://typeorm.io/#/insert-query-builder) y [delete](https://typeorm.io/#/delete-query-builder).


### Altas masivas - método insert
Los repositorios incluyen la operación `insert`, que recibe como parámetro una lista de objetos planos (o sea, que no necesitan ser instancias de las clases que modelan la entidad correspondiente), e implementan el alta de todos ellos _en una única sentencia INSERT_.  
P.ej. este código TypeScript
```typescript
await cityRepository.insert([
    { name: 'Yavi', province: 'Jujuy', population: 500 },
    { name: 'Rosario', province: 'Santa Fe', population: 2100000 },
    { name: 'Córdoba', province: 'Córdoba', population: 2200000 },
    { name: 'CABA', province: 'CABA', population: 3100000 },
    { name: 'Bell Ville', province: 'Córdoba', population: 25000 },
    { name: 'Río Cuarto', province: 'Córdoba', population: 190000 },
    { name: 'Barreal', province: 'San Juan', population: 5500 },
    { name: 'Londres', province: 'Catamarca', population: 2500 }
]);
```
se resuelve en esta sentencia SQL
```
 INSERT INTO "cities"("name", "province", "population") 
 VALUES ($1, $2, $3), ($4, $5, $6), ($7, $8, $9), ($10, $11, $12), ($13, $14, $15), 
 ($16, $17, $18), ($19, $20, $21), ($22, $23, $24) RETURNING "id" 
 -- PARAMETERS: 
 -- ["Yavi","Jujuy",500,"Rosario","Santa Fe",2100000,"Córdoba","Córdoba",2200000,
 -- "CABA","CABA",3100000,"Bell Ville","Córdoba",25000,
 -- "Río Cuarto","Córdoba",190000,"Barreal","San Juan",5500,"Londres","Catamarca",2500]
 ```
Esto resulta mucho más eficiente que generar sentencias `INSERT` separadas para cada entidad.

La operación `insert` se describe [en la API de Repository de la doc de TypeORM](https://typeorm.io/#/repository-api).

**Atención**  
La operación `insert` no tiene en cuenta las [cascadas](./typeorm-cascada) que pudieran estar definidas en la entidad. P.ej. si se usa `insert` para hacer un alta masiva de solicitudes de cuenta, no va a agregar los análisis crediticios, por más que se especifiquen en los objetos que se le envían al `insert`.


## Cache de query
TypeORM maneja el concepto de _cache de query_, o sea, que los resultados de una consulta se guarden temporariamente en algún lugar (la _cache_) que se supone de acceso más rápido que el tiempo que tarda repetir la query.  
De esta forma, para consultas que se realicen de forma muy frecuente, la mayor parte de las veces va a mejorar la eficiencia, porque va a resolver la consulta accediendo a la cacche.

Para activar la cache para un `find`, alcanza con incluir la opción `cache`. Hagámoslo para la consulta de sucursales, que ordenamos por código.
```typescript
async getAgencies(): Promise<Agency[]> {
    return await this.agencyRepository.find({ 
        cache: 120000,
        relations: ["city"],
        order: { code: "ASC" }
    });
}
```
Estamos suponiendo un escenario en que la cantidad de sucursales no pasa de unos cientos, lejos de los millones de sucursales que manejamos al trabajar con índices. 
Para consultas con millones de resultados no tiene sentido la cache, porque estaríamos recargando un mecanismo pensado para manejar volúmenes acotados de datos. Además ... si hay que realizar una consulta con esta cantidad de resultados en forma muy frecuente, estamos en problemas.

En las consultas que se describen mediante un QueryBuilder, se puede activar la cache mediante la operación `cache` de los mismos.
```typescript
const query = await agenciesRepository.createQueryBuilder('ag');
query.cache(120000);
/* ... definición de condiciones, joins y otros ... */
```

El valor `120000` que asociamos a la propiedad `cache`, es su _tiempo de expiración_, que se define en milisegundos. Cuento rápidamente cómo funciona esto
- Cuando se hace la consulta por primera vez, se agrega una entrada en la cache que incluye la consulta, el resultado, y un timestamp. 
- Cuando se repite la consulta, si se encuentra una entrada para la consulta en la cache, se compara el timestamp actual con el registrado. si el tiempo transcurrido es mayor al de expiración, entonces se _invalida_ la entrada. Se vuelve a hacer la consulta "real" y se almacena el resultado obtenido en la entrada de la cache, actualizando el timestamp.

El manejo de un tiempo de expiración es necesario para que la consulta incorpore las eventuales modificaciones, en nuestro caso p.ej. que se agregue una sucursal, o se le cambie el nombre a alguna. Se puede definir un valor de tiempo de expiración global en las opciones de conexión, y también indicar explícitamente que se invalide una entrada de cache (en nuestro caso, p.ej. cuando se agrega una sucursal). Los detalles están en la [página de la doc de TypeORM sobre cache](https://typeorm.io/#/caching/).

En la configuración por defecto, la cache se almacena en una tabla separada en la misma base de datos. Con lo cual el acceso mediante cache, también implica hacer una consulta a la base.  
Entonces ¿por qué tardaría menos? Porque se guarda el resultado en una sola fila de la tabla de cache, y por lo tanto se ahorra el trabajo necesario para resolverla, que incluye: obtener todas las filas (que podrían no estar contiguas en el disco), resolver los joins a otras tablas (en este caso la correspondiente a la entidad `City`), ordenar los resultados.  
TypeORM también permite que se configure el dispositivo de almacenamiento de la cache, para habilitar variantes que no necesiten acceder a la misma BD. En particular, TypeORM ya viene preparado para que la cache se almacene en Redis. Los detalles, en la [página de la doc de TypeORM sobre cache](https://typeorm.io/#/caching/).


## Traer sólo algunos datos de cada entidad
Para entidades/tablas que tengan muchos atributos/columnas, se puede indicar que el resultado de una consulta incorpore solamente algunos. Para eso se usa la opción (de `find`) / operación (de `QueryBuilder`) llamada `select`. 

Este es un ejemplo de búsqueda usando `find`, en el que queremos obtener, de las solicitudes pendientes, id y cliente.
```typescript
const applications = await applicationRepository.find({ 
    where: { status: Status.PENDING }, 
    select: ["id", "customer"]
});
```
El resultado es una lista de entidades que tienen valores únicamente para los atributos indicados.
```typescript
AccountApplication { id: 1, customer: 'Juana Molina' }
AccountApplication { id: 8, customer: 'Felicitas Guerrero' }
AccountApplication { id: 14, customer: 'Ana Bolena' }
AccountApplication { id: 15, customer: 'Inés Lasalle' }
```
Mostramos también la consulta SQL que genera TypeORM, donde se ve la relación directa entre la opción `select` del `find`, y la cláusula `SELECT` de la consulta.
```sql
SELECT "AccountApplication"."id" AS "AccountApplication_id", 
       "AccountApplication"."customer" AS "AccountApplication_customer" 
FROM "account_applications" "AccountApplication" 
WHERE "AccountApplication"."status" = $1 -- PARAMETERS: ["Pending"]
```

Veamos un ejemplo con QueryBuilder, de la misma consulta agregando el código de sucursal.
```typescript
const query = await applicationRepository.createQueryBuilder('aa');
query.leftJoin("aa.agency", "ag");
query.select(["aa.id", "aa.customer", "ag.code"])
query.where("aa.status = :status", { status: Status.PENDING })
const applications = await query.getMany();
```
El resultado respeta la estructura de entidades, incluyendo solamente los datos solicitados.
```typescript
[
  AccountApplication {
    id: 1, customer: 'Juana Molina', agency: Agency { code: '022' }
  },
  AccountApplication {
    id: 8, customer: 'Felicitas Guerrero', agency: Agency { code: '022' }
  },
  AccountApplication {
    id: 14, customer: 'Ana Bolena', agency: Agency { code: '077' }
  },
  AccountApplication {
    id: 15, customer: 'Inés Lasalle', agency: Agency { code: '078' }
  }
]
```

Si es conveniente manejar objetos planos en lugar de respetar la estructura de las entidades, simplemente podemos pedirle a TypeORM `getRawMany()` en lugar de `getMany()`
```typescript
const applications = await query.getRawMany();
```
obteniendo un resultado de esta forma
```typescript
[
  { aa_id: 1, aa_customer: 'Juana Molina', ag_code: '022' },
  { aa_id: 8, aa_customer: 'Felicitas Guerrero', ag_code: '022' },
  { aa_id: 14, aa_customer: 'Ana Bolena', ag_code: '077' },
  { aa_id: 15, aa_customer: 'Inés Lasalle', ag_code: '078' }
]
```

Usando la operación `select` y las opciones `getRawOne` o `getRawMany`, también podemos incluir operadores de agregación, como se indicó al describir las [opciones de búsqueda](./typeorm-busqueda).


## Para practicar
Agregar un endpoint `PATCH` que pase todas las solicitudes con menos de 3 `requiredApprovals` y que estén en estado `Analysing`, al estado `Accepted`.

Agregar un endpoint `POST` que reciba en el body una lista `[{ customer, creditLimit }]` y haga el alta masiva de los análisis crediticios correspondientes, usando la operación `.insert(/* ... */)` del repositorio correspondiente.

Agregar (tal vez en un nuevo módulo de ciudades) un endpoint `GET` que devuelva, dado el nombre de una provincia, las ciudades de esa provincia, para cada una id, nombre, cantidad de sucursales, y superficie total de las sucursales.
