---
layout: default
---

# Mapeo de relaciones
Una de las características principales de las BD relacionales son (justamente) las herramientas que ofrece, orientadas a establecer, usar y mantener _relaciones_ entre entidades. Entre estas herramientas encontramos: el concepto de FK, las restricciones de integridad, las operaciones en cascada.

TypeORM incluye herramientas para especificar relaciones, y para usarlas. Estas herramientas hacen que algunas correspondencias entre el código y la base sean menos rígidas. En particular
- en las clases que se mapean a tablas, van a aparecer atributos con decorators de TypeORM, para los cuales no se define una columna en la tabla correspondiente.
- las operaciones que se realizan mediante un repositorio, pueden afectar a otras tablas, además de la relacionada con la entidad del repositorio.


## Relaciones many-to-one
Agreguemos las _sucursales_ como un segundo tipo de entidad, al modelo que hasta ahora tiene solamente las solicitudes de cuenta. 
```typescript
@Entity({ name: 'agencies' })
export class Agency {
    @PrimaryGeneratedColumn()
    id: number

    @Column({ length: 10 })
    code: string

    @Column({ length: 120 })
    name: string

    @Column({ nullable: true })
    address: string

    @Column({ nullable: true })
    area: number
}
```

Queremos asociar cada solicitud de cuenta con la sucursal donde se generó, en el programa y también en la BD.

En el código, alcanza con agregar un atributo en la clase `AccountApplication`.
```typescript
@Entity({ name: 'account_applications' })
export class AccountApplication {
    // ... otros atributos ...

    agency: Agency
```

En la BD, debe establecerse una **relación** entre las entidades `account_application` y `agency`. 
Mirado desde las solicitudes, esta relación es del tipo _muchos-a-uno_, o _many-to-one_, porque:
- puede haber _muchas_ solicitudes relacionadas con una misma sucursal, pero
- cada solicitud está relacionada con _una sola_ sucursal.

El decorator `@ManyToOne` de TypeORM define una relación de este tipo. Queda así:
```typescript
@Entity({ name: 'account_applications' })
export class AccountApplication {
    // ... otros atributos ...

    @ManyToOne(() => Agency)
    agency: Agency
}
```
El parámetro le indica a TypeORM cuál es la clase que mapea la entidad relacionada. Por razones que desconozco, se define como una función.

Esta relación se implementa en la BD, agregando una columna en la tabla `account_applications` cuyo valor es _la primary key_ (el id numérico por como venimos definiendo las PK) de la sucursal asignada. Esta columna se marca en la BD como una FK a la tabla `agencies`. 
Si en las opciones de conexión tenemos `synchronize = true`, entonces al levantar el programa o aplicación, TypeORM va a crear la columna y la FK. En este ejemplo, agrega a la tabla `account_applications` la columna `agencyId`.

Para las solicitudes que se agreguen, es conveniente asignarles una sucursal obtenida desde la BD:
```typescript
const agency22 = await agencyRepository.findOneOrFail({ code: '022' });

const newApplication = new AccountApplication();
newApplication.customer = "Felicitas Guerrero";
newApplication.status = Status.PENDING;
newApplication.requiredApprovals = 3;
newApplication.agency = agency22;
const savedApplication = await applicationRepository.save(newApplication);
```

En la nueva fila de `account_applications`, el valor de `agencyID` es el `id` de la sucursal asignada.


## Relaciones al obtener datos
Hagamos una búsqueda cuyo resultado sea la solicitud recién creada.
```typescript
await applicationRepository.findOneOrFail({ customer: "Felicitas Guerrero" });
```
El resultado va a tener esta forma
```typescript
AccountApplication {
  id: 8,
  customer: 'Felicitas Guerrero',
  status: 'Pending',
  date: null,
  requiredApprovals: 3
}
```
... la sucursal que le asignamos ¡no aparece!

La razón es que en principio, cuando TypeORM obtiene una entidad desde la BD, no incluye a las entidades relacionadas. 
Tal como mencionamos en la introducción a TypeORM, el motivo está relacionado con la performance: obtener todos los objetos relacionados, directa o indirectamente, con el objeto que se está buscando, podría provocar un query con un resultado enorme, que implica mucho cálculo en la BD y mucho tráfico entre la BD y el programa / servicio / aplicación. 

Para que el resultado de un query incluya entidades relacionadas con las que se están buscando, hay que mencionarlo explícitamente usando la opción `relations`. Las opciones de búsqueda se definen por separado de los criterios (en el ejemplo que sigue, el criterio es el cliente, y la opción es `relations`). Una opción es pasar un objeto con las opciones, donde los criterios se definen como una opción llamada `where`.
```typescript
await applicationRepository.findOneOrFail({ 
    where: { customer: 'Felicitas Guerrero' }, 
    relations: ["agency"] 
});
```

El resultado de esta query sí va a incluir la sucursal.
```typescript
AccountApplication {
  id: 8,
  customer: 'Felicitas Guerrero',
  status: 'Pending',
  date: null,
  requiredApprovals: 3,
  agency: Agency {
    id: 5,
    code: '022',
    name: 'Yavi',
    address: 'Av. San Martín 42',
    area: null
  }
}
```

Notamos que en este query sobre el repositorio de solicitudes, se integra la tabla de solicitudes con otras tablas. Por lo tanto, al hacer queries sobre un repositorio, el resultado va a ser información que esté en la tabla correspondiente ... y en las relacionadas.


### Cadenas de relaciones
Supongamos ahora que agregamos una entidad `City`, y asignamos la ciudad en la que está cada sucursal.
```typescript
@Entity({ name: 'agencies' })
export class Agency {
    // ... otros atributos ...

    @ManyToOne(() => City)
    city: City;
}
```

Para incluir los datos de la agencia _y los de la ciudad_ en un query, podemos especificar la búsqueda de esta forma.
```typescript
await applicationRepository.findOneOrFail({ 
    where: { customer: 'Felicitas Guerrero' }, 
    relations: ["agency", "agency.city"] 
});
```
obteniendo el siguiente resultado
```typescript
AccountApplication {
  id: 8,
  customer: 'Felicitas Guerrero',
  status: 'Pending',
  date: null,
  requiredApprovals: 3,
  agency: Agency {
    id: 5,
    code: '022',
    name: 'Yavi',
    address: 'Av. San Martín 42',
    area: null,
    city: City { id: 1, name: 'Yavi', province: 'Jujuy', population: 500 }
  }
}
```


## El otro lado de las relaciones
Con el código y los mapeos TypeORM como están, para obtener las solicitudes asignadas a una sucursal, hay que hacer una consulta sobre solicitudes. 
```typescript
const agency22 = await agencyRepository.findOneOrFail({ code: '022' });
const applications = await applicationRepository.find({ 
    where: { agency: agency22 },
    relations: ["agency"] 
});
``` 

No podemos acceder a este conjunto de solicitudes a partir de la sucursal.  
Dicho de otra forma, la relación es de acceso _unidireccional_: podemos acceder a esta relación sólo en una dirección, desde las solicitudes hacia las sucursales, pero no al revés. 

Para habilitar el acceso _bidireccional_ a esta relación, en el código deberíamos agregar un atributo en `Agency`, cuyo valor es _una lista_ de solicitudes:
```typescript
@Entity({ name: 'agencies' })
export class Agency {
    // ... otros atributos ...

    accountApplications: AccountApplication[]
}
```
Obviamente, es responsabilidad nuestra mantener la coherencia entre este atributo, y el valor del atributo `agency` en cada solicitud.

Para que TypeORM pueda manejar la relación en forma bidireccional, tenemos que agregarle a este atributo el decorator `@OneToMany`, y agregar en los cada uno de los extremos, la forma de acceder a la relación en el otro. Se ve más fácil en el código.
```typescript
@Entity({ name: 'account_applications' })
export class AccountApplication {
    // ... otros atributos ...

    @ManyToOne(() => Agency, agency => agency.accountApplications)
    agency: Agency
}

@Entity({ name: 'agencies' })
export class Agency {
    // ... otros atributos ...

    @OneToMany(() => AccountApplication, app => app.agency)
    accountApplications: AccountApplication[]
}
```
