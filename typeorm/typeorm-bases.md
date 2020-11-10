---
layout: default
---

# Las bases de TypeORM

Para configurar el uso de TypeORM en una aplicación, hay que tener en cuenta tres aspectos: _conexión_, _modelo de datos_ y _operaciones_. 

Al igual que en Mongoose, 
- se necesita establecer una conexión con la/s BD con las que se quiera interactuar, y
- las operaciones son las de CRUD: alta, búsqueda, modificación, eliminación.

Como diferencias con Mongoose (al menos en una primera mirada) subrayamos
- no existen dos conceptos separados de _esquema_ y _modelo_, el modelo de datos se describe a partir de clases usando decorators, en forma similar a como describimos validaciones de `class-validator` / `ValidatorPipe`.
- no existe una clase análoga al `Document` de Mongoose, los objetos que contienen los datos son planos. Por eso, todas las operaciones se las vamos a pedir a objetos específicos para describir operaciones, no a los objetos que contienen los datos.  
Una consecuencia es que los objetos que nos traigamos de la base van a ser más "livianos", no tiene sentido una operación como el `toObject()` de Mongoose, lo que se obtiene de la base ya es un "toObject".


## Antes de arrancar
Hay que incorporar el package `typeorm`, más el _driver_ del motor de BD que querramos usar.
El _driver_ es una librería de bajo nivel para interactuar con una BD. Según el motor que usemos (MySQL, Postgres, SQLServer, Oracle, etc.) vamos a necesitar un driver distinto.  
Además, TypeORM pide incorporar un package adicional llamado `reflect-metadata`, que tenemos que importar en algún lado (p.ej. en el `index`). 

En resumen, para usar TypeORM con una BD Postgres, hay que hacer lo siguiente (si se usa npm):
``` 
npm install --save typeorm reflect-metadata pg
```
Para MySQL, reemplazar `pg` por `mysql`.

Se pueden ver los detalles en el ["getting started" de la doc de TypeORM](https://typeorm.io/#/).


## Conexión
Para crear una conexión, alcanza con invocar a la función `createConnection` que viene con el package `typeorm`.

Los parámetros de conexión se pueden especificar, o bien como un objeto que se le pasa por parámetro a `createConnection`, o bien en un archivo separado, p.ej. `ormconfig.json` que esta función sabe dónde ir a buscar.

Este sería un ejemplo de la primer variante.

```typescript
import { createConnection } from "typeorm";

const connection = await createConnection({
    type: "postgres",
    host: "localhost",
    port: 5432,
    username: "postgres",
    password: "notapassword",
    database: "nano_bank_model",
    synchronize: true,
    logging: false,
    entities: [
        "src/entity/**/*.ts"
    ]   
});
```
En este caso nos estamos conectando a la base `nano_bank_model` de un Postgres (ese es el `type`) que corre en localhost.

La alternativa sería tener un archivo `ormconfig.json` en la carpeta raíz del proyecto, de esta forma
```json
{
   "type": "postgres",
   "host": "localhost",
   "port": 5432,
   "username": "postgres",
   "password": "rank3041",
   "database": "nano_bank_model",
   "synchronize": true,
   "logging": false,
   "entities": [
      "src/entity/**/*.ts"
   ],
 }
```

e invocar directamente
```typescript
const connection = await createConnection()
```
Hay más variante sobre cómo especificar los parámetros de conexión, ver [la página sobre archivos con parámetros de conexión en la doc](https://typeorm.io/#/using-ormconfig).


Nos quedamos con una referencia a la conexión, porque le vamos a pedir los objetos necesarios para realizar operaciones, y al final tenemos que cerrarla
```typescript
try {
    // operaciones
} catch (e) {
    // manejo de error
} finally {
    await connection.close();
}
```

Más información en [la página sobre establecer y usar conexiones en la doc](https://typeorm.io/#/connection). TypeORM admite trabajar con [múltiples conexiones](https://typeorm.io/#/multiple-connections).


### Información adicional de conexión
Vemos que el objeto que le llega a `createConnection` incluye información adicional a los parámetros de conexión (o sea host/port, user/pwd, base). TypeORM llama a toda esta info, que incluye a los parámetros, _opciones de conexión_.
Específicamente, encontramos tres opciones adicionales: `synchronize`, `logging` y `entities`. Las describimos ... en orden inverso.

En `entities` especificamos dónde tiene que buscar TypeORM los modelos de datos, o sea las clases con decorators que van a representar en el programa, el modelo de la información que está en la base.

Si "prendemos" `logging`, va a aparecer por consola **toooodo** el SQL que genera TypeORM para resolver las operaciones que le pidamos. Sí, se puede redefinir _dónde_ loguea y (un poquito) también qué loguea. Los detalles en [la página sobre logging en la doc](https://typeorm.io/#/logging/).

Finalmente, si tenemos `synchronize` "prendido", entonces cada vez que levantemos el programa/app, va a sincronizar el esquema de la BD de acuerdo a la especificación del modelo de datos. 
P.ej. si agregamos una clase al modelo de datos que se corresponde con una tabla, la tabla no está definida en la BD a la que nos estamos conectando, y tenemos `synchronize` en `true`, entonces al levantar el programa/app va a crear la tabla.
Esto **no** debería estar prendido en producción. 

TypeORM define muchas otras opciones, algunas generales y otras específicas para motores de DB determinados (algunas opciones sólo para MySQL, otras sólo para Postgres, etc.). El detalle está en [la página sobre opciones de conexión en la doc](https://typeorm.io/#/connection-options).


## Modelo de datos
La conexión indica qué base vamos a usar. En principio, cada tabla en esa base que querramos manejar dentro del programa la vamos a describir, o _mapear_, mediante una clase, a la que aplicamos decorators propios de TypeORM.  
**Además**, la clase tiene que estar definida en un archivo dentro de los paths definidos para las `entities` en las opciones de conexión.

Creo que se cuenta más fácil con un ejemplo: para describir una tabla que va a persistir solicitudes de cuenta, podemos definir una clase como sigue.  
```typescript
@Entity({ name: 'account_applications' })
export class AccountApplication {
    @PrimaryGeneratedColumn()
    id: number

    @Column({ length: 120 })
    @IsDefined()   // ¡¡ este es de class-validator !!
    customer: string

    @Column({ type: "enum", enum: Status, default: Status.PENDING })
    status: Status

    @Column({ nullable: true })
    date: number

    @Column()
    requiredApprovals: number

    @Column({ type: "decimal", precision: 15, scale: 2})
    estimatedBalance: number
}
``` 

Mediante el decorator de clase `@Entity`, indicamos que la clase mapea una tabla. Entre las opciones de este decorator está `name`, ahí podemos indicar el nombre de la tabla. Si no lo hacemos, TypeORM definirá un nombre de tabla por defecto.

El decorator de atributo `@Column` sirve para mapear una columna de la tabla.  
- El _tipo de datos_ de la columna se puede especificar, o no. Si no se especifica, TypeORM va a asumir ciertos tipos básicos de acuerdo al tipo del atributo: va a mapear `string` como `varchar`, `number` como `int4`, `boolean` como `boolean`. En el ejemplo, dejamos los tipos por defecto en tres atributos, y definimos explícitamente los tipos `enum` y `decimal` para los otros dos.   
- Este decorator tiene opciones que corresponden a los _parámetros para especificar una columna_ en la BD, p.ej. la opción `nullable` es el "not null" de la BD, el `length` en un atributo string es el length del `varchar`. Algunas opciones son generales, otras son específicas para uno o varios motores de BD, p.ej. `enum` vale solamente para Postgres y MySQL.

La descripción de cómo vamos a mapear una tabla debe incluir cómo se va a manejar su _clave primaria_, para les amigues PK.  
TypeORM define decorators para distintas variantes. En el ejemplo elegimos la misma que se usa en varios servicios del Banco, una clave numérica autoincremental, o sea que el valor es asignado por la misma base al hacer el INSERT. Esta opción se describe mediante el decorator `@PrimaryGeneratedColumn`.

Los detalles del mapeo de una entidad, incluyendo tooodas las opciones de `@Entity` y `@Column`, los distintos tipos de columna que puede manejar TypeORM, y las variantes para definir/mapear la PK, están detallados en la [página sobre entidades de la doc](https://typeorm.io/#/entities).


### Quién maneja el esquema de BD
Si tenemos "prendida" la opción `synchronize`, entonces TypeORM ajusta el esquema de BD al mapeo que hayamos definido. O sea, si hay una entidad mapeada y la tabla no está en la BD, entonces TypeORM crea la tabla al levantar la app, programa o script. También agrega las columnas que estén mapeadas a atributos, pero no estén en la tabla. Por esta razón, es cómodo tener `synchronize: true` en desarrollo, un poco nos olvidamos de manejar el esquema de la BD.  
Si `synchronize` está "apagada", entonces TypeORM confía en que el esquema de BD esté correctamente definido. Si hay inconsistencias, pueden generarse resultados incorrectos al realizar operaciones, hasta donde veo no chequea el esquema. No encontré una opción para que TypeORM chequee el esquema de BD al levantar. 

**Nota**: No sé si la sincronización es perfecta, o sea que cualquier cosa que se cambie en la definición de cada columna va a impactar en la tabla.  

Para manejar los cambios en una BD que se introducen al actualizar un programa, TypeORM incorpora el concepto de [migraciones](https://typeorm.io/#/migrations).


### Validaciones de los datos a incorporar a la BD
A diferencia de p.ej. Mongoose, TypeORM no incorpora validaciones. Las definiciones que se incluyen en los mapeos, p.ej. las columnas que se generen con `nullable: false`, se traducen como restricciones en la definición del esquema de BD. 
A partir de estas restricciones, la BD puede detectar datos erróneos. En tal caso la operación de BD sale con una excepción, que TypeORM transforma en una `QueryFailedError`.

La doc de TypeORM incluye una [página sobre validaciones](https://typeorm.io/#/validation) ... en la que se recomienda el uso de `class-validator`, sin dar un soporte específico (como el de `ValidationPipe` de Nest), es el programa quien tiene que invocar a la función `validate` y procesar su resultado.



## Operaciones
TypeORM ofrece distintas alternativas para realizar operaciones sobre la BD. En algunos casos se debe a una cuestión de estilo, en otros, hay variantes que están más cerca de la BD, que pueden ser más incómodas para usar pero dan más performance, y otras más cómodas que pueden resultar más lentas.

En esta página vamos a describir _una_ forma de especificar operaciones, que va en la línea de cómo se usa TypeORM en servicios del banco. Más adelante veremos algunas de las alternativas ... para verlas todas, nos remitimos a la documentación.


### Repository - acceso a una entidad
Una de las herramientas que ofrece TypeORM para realizar operaciones son los llamados _repositorios_. Un repositorio permite interactuar con una entidad, que en el modelo es una clase que tiene el decorator `@Entity`, incluyendo todas las operaciones CRUD.

Para obtener un repositorio, hay que pedírselo a la conexión.
```typescript
const applicationRepository = connection.getRepository(AccountApplication);
```
A partir del `applicationRepository`, podremos crear, obtener, modificar y eliminar solicitudes de cuenta.

A continuación describimos _algunas_ operaciones que se pueden hacer con un repositorio, a modo de ejemplo. Se puede consultar toda la interfaz [en la página correspondiente de la doc](https://typeorm.io/#/repository-api).


### Retrieval - traernos datos de la base
Para obtener información de la base, en nuestro ejemplo las solicitudes de cuenta que estén persistidas, usamos la operación `find` de los repositorios.
```typescript
const applications: AccountApplication[] = await repository.find();
```
... obviamente, todas las operaciones de BD son asincrónicas ... indicamos el tipo de `applications` para que

La operación `find` admite parámetros para especificar condiciones de búsqueda y entidades relacionadas que se quieran incorporar a la consulta. Trataremos cada uno de estos dos temas, más adelante.

También hay operaciones que son variantes del `find`: `findByIds`, `findOne`, `findOneOrFail`, `findAndCount`.


### Save - alta o modificación
La operación `save` de los repositorios realiza lo que se conoce como "upsert": si la entidad ya existe en la base la modifica, si no existe la agrega. 

Esta es una forma sencilla de hacer un alta.
```typescript
const newApplication = new AccountApplication();
newApplication.customer = "Perdita Durango";
newApplication.status = Status.REJECTED;
newApplication.requiredApprovals = 4;
const savedApplication = await applicationRepository.save(newApplication);
```

Este es un caso sencillo de modificación: obtenemos el objeto a modificar (en este caso suponemos que conocemos el id), cambiamos lo que querramos, `save`. En este caso, aumentamos en uno la cantidad de aprobaciones requeridas para aceptar la solicitud.
```typescript
const application = await repository.findOneOrFail({ id: 3 });
application.requiredApprovals++;
await repository.save(application);
```

Los repositorios también ofrecen operaciones para abreviar el código de creación del objeto al que se le va a hacer `save`. P.ej. tenemos `create`, lo mostramos con un ejemplo que es una forma alternativa de hacer la misma alta que mostramos recién.
```typescript
const newApplication = applicationRepository.create(
    { customer: "Perdita Durango", status: Status.REJECTED, requiredApprovals: 4 }
);
const savedApplication = await applicationRepository.save(newApplication);
```
Esta variante puede ser cómoda p.ej. cuando los datos vienen en el body de un request. Otras operaciones que pueden verse [en la doc](https://typeorm.io/#/repository-api) son `merge` y `preload`.



---
**Nota**  

¿Cómo se da cuenta el repositorio si la entidad existe o no?  
Respuesta: por el valor de la PK. Si ya hay una entidad con esa PK la modifica, si no la agrega.
O sea ... que tiene que hacer un `SELECT` para decidir si va un `INSERT` o un `UPDATE` ... más cómodo, pero más lento.  
Por suerte, si la entidad _no tiene seteado_ valor para la PK, entonces TypeORM decide hacer un `INSERT`, sin necesidad de hacer un `SELECT` previo.

---


### Comentario final 1
En principio, todas las operaciones que realicemos mediante un repositorio van a interactuar con una única tabla en la BD ... aunque esto puede cambiar de acuerdo a cómo modelemos las _relaciones_.


### Comentario final 2 - ActiveRecord o DataMapper
Comparemos el alta como la implementamos en TypeORM
```typescript
const newApplication = new AccountApplication();
// ... se completan los datos ...
const savedApplication = await applicationRepository.save(newApplication);
```
con la misma operación en Mongoose
```typescript
const newApplication = new AccountApplicationModel();      
// ... se completan los datos ...
newApplication.save();
```

En Mongoose, algunas operaciones, en particular `save`, se le piden al documento. En cambio, de la forma en que estamos usando TypeORM, las operaciones siempre se le piden al repositorio.  
Esto muestra dos formas de trabajar con una BD desde una aplicación, a las que se asocian los nombres **active record** y **data mapper**. Mongoose trabaja con active records.

En rigor TypeORM _puede trabajar en cualquiera de los dos modos_, como lo cuenta en [una página específica de la doc](https://typeorm.io/#/active-record-data-mapper). En los servicios del banco, y en este material, se trabaja con data mappers. Los repositorios son data mappers.

Una consecuencia de esta diferencia en cómo trabajamos en Mongoose y en TypeORM, es que los documentos de Mongoose son objetos que además de los datos de la BD, incluyen mucha más información. Por eso está la operación `toObject()` que permite obtener el objeto plano. 
Trabajando con data mappers, en TypeORM directamente obtenemos objetos planos, no tiene sentido un `toObject()`.


