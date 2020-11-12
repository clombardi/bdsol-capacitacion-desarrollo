---
layout: default
---

# Cascada
Consideremos estos dos tipos de entidad, que tienen una relación bidireccional.
```typescript
@Entity({ name: 'agencies' })
export class Agency {
    @PrimaryGeneratedColumn()
    id: number

    // ... varios atributos ...

    @OneToMany(() => OfficeParty, party => party.agency)
    parties: OfficeParty[]
}

@Entity({ name: 'office_parties' })
export class OfficeParty {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    peopleAttending: number

    @Column({ type: "decimal", precision: 15, scale: 2 })
    budget: number

    @ManyToOne(() => Agency, agency => agency.parties)
    agency: Agency
}
```

Esta relación representa que en una sucursal se pueden hacer varias fiestas, y que cada fiesta corresponde a una sola sucursal.


## Operaciones sobre entidades relacionadas
Supongamos que queremos agregar una sucursal con dos fiestas ya cargadas. Aprovechando el método `create` de los repositorios para ahorrar código, podríamos armar algo así.
```typescript
const newAgency = agencyRepository.create({
    code: '204', name: 'Cruz del Eje', address: 'Dr. Illia 28', area: 231
});
newAgency.parties = [
    partyRepository.create({ peopleAttending: 88, budget: 9300, agency: newAgency }),
    partyRepository.create({ peopleAttending: 101, budget: 180000, agency: newAgency })
];
await agencyRepository.save(newAgency);
```

Tal vez esperaríamos que este código agregue a la base, una sucursal y dos fiestas. Con los mapeos como están, esto no va a ocurrir. Si después levantamos la sucursal de la base, obtenemos esto.
```typescript
Agency {
  id: 10,
  code: '204',
  name: 'Cruz del Eje',
  address: 'Dr. Illia 28',
  area: 231,
  parties: []
}
```

¿Qué pasó?  
Que _en principio_, **las operaciones sobre una entidad no se propagan a sus entidades relacionadas**.
En este caso, estamos agregando una sucursal, TypeORM va a (generar el SQL para) agregar la sucursal, y nada más. Las fiestas son entidades aparte, no se agregan.

En el dominio de las bases de datos relacionales, se usa la palabra _cascada_ para mencionar la propagación de una operación en una entidad, sobre entidades relacionales. Lo que vimos en el ejemplo es que TypeORM no ejecutó la cascada del alta de sucursal hacia las fiestas que se le asignaron.

Hay distintos casos de operaciones que podrían propagarse. 

El que vimos está relacionado con el _alta de la "entidad principal"_, o sea de la sucursal. 

Hay varias variantes que son _modificaciones de la entidad principal_, las resumimos en este extracto de código.
```typescript
/* agregar una fiesta (o varias) a la sucursal */
agency.parties.push(partyRepository.create({ peopleAttending: 387, budget: 8300, agency }));

/* eliminar una, o varias, fiesta/s de la lista de fiestas de la sucursal */
remove(agency.parties, party => party.peopleAttending === 100);
// la función remove es cortesía de lodash

/* modificar datos de una fiesta accediéndola desde la sucursal */
agency.parties[0].peopleAttending++;

/* en cualquier caso, se registra la operación desde la sucursal */
await agencyRepository.save(agency);
```

El último caso es la _eliminación de una entidad principal_, o sea de una sucursal, que tiene fiestas cargadas. 
¿Qué esperaríamos que ocurriera con las fiestas?  
Notemos que este caso tiene una característica distinta a los anteriores: no estamos modificando la lista de fiestas de la sucursal.


## Cascada de TypeORM
TypeORM incluye una opción `cascade` en la definición de las relaciones. Las opciones se incluyen como un argumento adicional en los decorators que definen las relaciones.
```typescript
@Entity({ name: 'agencies' })
export class Agency {
    @PrimaryGeneratedColumn()
    id: number

    // ... varios atributos ...

    @OneToMany(() => OfficeParty, party => party.agency, { cascade: true })
    parties: OfficeParty[]
}
```

Esta opción cubre los casos en que se modifica algún objeto relacionado, y en el caso de `@OneToMany`, si se modifica (o se define) la colección. 
En los ejemplos de la sección anterior, todos los casos _excepto la eliminación de una sucursal_.
Al agregarse una sucursal con fiestas cargadas, o bien agregar/eliminar/modificar fiestas de una sucursal, la operación efectuada sobre la sucursal se va a propagar a las fiestas ...

... aunque **atención**, si se quita una fiesta de la sucursal, la fila en `office_parties` no se elimina, sino que su `agencyId` queda en `NULL` ... o sea, para eliminar, mejor hacerlo desde el repositorio de fiestas.


### Cascadas de los dos lados
La cascada se puede poner en cualquiera de las dos direcciones de una relación. Si _en lugar_ de agregarla en la sucursal, la indicamos en la fiesta.

```typescript
@Entity({ name: 'office_parties' })
export class OfficeParty {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    peopleAttending: number

    @Column({ type: "decimal", precision: 15, scale: 2 })
    budget: number

    @ManyToOne(() => Agency, agency => agency.parties, { cascade: true })
    agency: Agency
}
```

se van a propagar operaciones de una fiesta hacia su sucursal, p.ej.
```typescript
const party = await partyRepository.findOneOrFail({
    where: { peopleAttending: 88 }, relations: ["agency"]
});
party.agency.area = 804;
await partyRepository.save(party);
```

---
**Nota** sobre sutilezas de la cascada de TypeORM  

TypeORM no permite habilitar `{ cascade: true }` en **ambos sentidos** de una relación bidireccional, porque dice que podría darse una circularidad de los `delete`.  
Pero sí se puede hacer, si se _restringe_ la cascada a alta y modificación, o sea, en un sentido cambiar el `{ cascade: true }` por `{ cascade: ["insert", "update"] }`.  
O sea, se puede restringir la cascada a algunas operaciones. 

No encontré documentación buena sobre cascadas, la única explicación, sobre una versión vieja, está [en esta página](https://orkhan.gitbook.io/typeorm/docs/relations). 

---


## Cascada de la BD
La cascada de una entidad a sus relacionadas en el caso de eliminación de la entidad principal, la manejan **las BD relacionales**. 
Si no se especifica nada, la BD va a impedir eliminar una entidad que tenga otras relacionadas, porque estas últimas quedarían "huérfanas". En el ejemplo, no va a permitir eliminar una sucursal que tenga fiestas cargadas, porque ... el valor de la columna `agencyId` en `office_parties` sería la PK de una sucursal ... que no existe más. Y este es, exactamente, el tipo de inconsistencias que una BD relacional evita.

Se le puede pedir a la BD que la eliminación se propague a las entidades relacionadas, borrándolas también. Esa opción se llama `ON DELETE CASCADE` (en mayúscula como le gusta a SQL), y se pone _en la definición de la FK_. 

TypeORM permite especificar que cuando cree la tabla, le agregue esta opción a la FK. Esto se indica como una opción de la relación, que _hay que poner del lado en que se va a generar la columna_. En una relación muchos-a-uno, del lado que es `@ManyToOne`. En nuestro ejemplo.

```typescript
@Entity({ name: 'office_parties' })
export class OfficeParty {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    peopleAttending: number

    @Column({ type: "decimal", precision: 15, scale: 2 })
    budget: number

    @ManyToOne(
        () => Agency, agency => agency.parties, 
        { onDelete: 'CASCADE', cascade: ["insert", "update"] }
    )
    agency: Agency
}
```

Como se ve, se pueden combinar las cascadas de la BD con las de TypeORM.
Igualmente, el caso más común es poner el `cascade: true` de TypeORM en un extremo de la relación, y el `onDelete: 'CASCADE'` de la BD en el otro.



## Para practicar
Si se elimina una fiesta desde el repositorio de fiestas, ¿se elimina de la lista de fiestas de la sucursal?
Separar en dos escenarios
- ya tengo la sucursal levantada
- voy a buscar la sucursal después de eliminada la fiesta.

En la relación entre solicitudes de cuenta y análisis de crédito, establecer la cascada del lado de las solicitudes. Probar que andan los casos de alta de una solicitud con análisis cargado, de cargarle un análisis a una solicitud que no tenía, y de modificar datos del análisis desde la solicitud.  
Probar qué pasa si se la asigna un análisis a una solicitud que ya tenía uno cargado.

En esta misma relación ¿tiene sentido poner `onDelete: 'CASCADE'` en algún extremo de la relación? ¿En cuál? ¿Qué efecto tiene?

Las cascadas de la BD tienen otra opción: `ON DELETE SET NULL`. ¿Se puede especificar en TypeORM? ¿Qué efecto tiene?