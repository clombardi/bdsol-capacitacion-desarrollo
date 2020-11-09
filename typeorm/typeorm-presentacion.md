---
layout: default
---

# Presentamos a TypeORM

Para ampliar el material sobre cómo interactuar con una base de datos desde un backend, llega el momento de experimentar con una librería del tipo _ORM_, o sea, que está orientada a la interacción con una base de datos relacional.

Para esto vamos a usar [TypeORM](https://typeorm.io/), un ORM que está bien preparado para incorporarlo a un proyecto TypeScript, aunque también se puede usar en JavaScript.  
Hay muchos otros ORM para JS/TS, entre ellos mencionamos [Sequelize](https://sequelize.org/), que es anterior y más popular ... pero a juicio de este redactor, más incómodo para usar, o tal vez menos claro, que TypeORM.


## Por qué queremos un ORM

Una BD relacional está **mucho** más lejos de un lenguaje como JS o TS que una base de documentos, en particular de Mongo. Destacamos dos razones para esto.  

La primera es que el **modelo de datos relacional** lleva a distribuir en varias tablas, información que en el modelo de un programa podría estar agrupada en un único objeto. 
Por ejemplo, la información sobre personas con varias direcciones se podría representar usando un único objeto JS/TS con esta estructura:
``` typescript
[
    {
        nombre: 'Juana',
        apellido: 'Molina', 
        cuil: '23-88442299-1',
        direcciones: [
            { calle: 'Yavi', numero: 328 }, { calle: 'Olinda', numero, 42 }
        ]
    },
    {
        nombre: 'Nelly',
        apellido: 'Omar', 
        cuil: '27-17101945-4',
        direcciones: [
            { calle: 'Eva Perón', numero: 745 }, { calle: 'Ramón Carrillo', numero: 901 }
        ]
    },

```

En una BD relacional, para representar la misma información necesitamos dos tablas, una con los datos "principales" de la persona, y otra con las direcciones. A su vez, estas dos tablas tienen que estar vinculadas, por lo tanto en la tabla de direcciones tendremos una FK a la tabla de personas.

La tabla de personas podría tener esta forma. 

| id | nombre | apellido | cuil |
| --- | --- | --- | --- |
| 1 | Juana | Molina | 23-88442299-1 |
| 2 | Nelly | Omar | 27-17101945-4 |

Y así podría ser la tabla de direcciones, donde `personaId` es una FK a la tabla de personas.

| id | calle | numero | personaId |
| --- | --- | --- | --- |
| 11 | Yavi | 328 | 1 |
| 12 | Olinda | 42 | 1 |
| 13 | Eva Perón | 745 | 2 |
| 14 | Ramón Carrillo | 901 | 2 |

La segunda razón que hace incómodo trabajar con una BD relacional es que la **sintaxis** de SQL es muy distinta a la de JS/TS (y de la mayoría de los lenguajes actuales). Veremos un ejemplo concreto en breve.

Un ORM ayuda a interactuar con una BD relacional, sin que sea necesario desviarnos mucho de la lógica y las estructuras que manejamos en un programa. Un _buen_ ORM debería (arriesgo) apuntar (al menos) a dos cosas:
1. en general, permitir que expresemos nuestro modelo y nuestras operaciones con una sintaxis y una lógica coherentes con el programa que estamos armando, que no sea "SQL disfrazado de".
1. brindar herramientas que nos permitan trabajar "más cerca" de la BD relacional, en los casos en que esto se requiera por cuestiones de complejidad de queries y/o performance.


## TypeORM a grandes rasgos

TypeORM propone una descripción de la interacción con una BD basada fuertemente en conceptos y elementos de JS/TS.

Para describir el **modelo de datos** se utilizan clases JS/TS (como lo indicamos para otras herramientas, no se pueden usar interfaces porque no están presentes en el código que se ejecuta) a las que se aplican decorators, en forma muy similar a la de `class-validator`. Básicamente, `@Entity` representa una tabla, y `@Column` una columna.  
Como TypeORM está orientado al modelo relacional, _tiene muy en cuenta las relaciones entre tablas_: define decorators específicos para estas relaciones (en concreto, `@OneToOne`, `@OneToMany`, `@ManyToOne` y `@ManyToMany`), e incluye herramientas que generan y manejan automáticamente las FK.

Para describir las **operaciones**, se utilizan varios objetos que provee la biblioteca. En particular, en este material vamos a acceder a las entidades mediante los llamados `Repository`, que proveen el acceso a una tabla, o sea, un `Repository` va a estar asociado a una clase que hayamos decorado como `@Entity`.

P.ej. una de las formas que provee TypeORM para describir un query es la operación `findOne` que soportan los `Repository`.  
El modelo con el que vamos a trabajar, incluye solicitudes de cuenta, sucursales y análisis de riesgo crediticio; cada solicitud corresponde a una sucursal y puede motivar la realización de un análisis. Para obtener la información acerca de la solicitud de Ana Bolena, en SQL debemos realizar un query de esta forma.

``` sql
SELECT aa.id AS aa_id, aa.customer AS aa_customer, aa.status AS aa_status, 
       aa.date AS aa_date, aa."requiredApprovals" AS aa_requiredApprovals, 
       aa."agencyId" AS aa_agencyId, aa."creditAssessmentId" AS aa_creditAssessmentId, 
       ag.id AS ag_id, ag.code AS ag_code, ag.name AS ag_name, ag.address AS ag_address, 
       ag.area AS ag_area, ca.id AS ca_id, ca.customer AS ca_customer, 
       ca."creditLimit" AS ca_creditLimit 
FROM account_applications aa 
LEFT JOIN agencies ag ON ag.id=aa."agencyId" 
LEFT JOIN credit_assessments ca ON ca.id=aa."creditAssessmentId"
WHERE aa.customer = 'Ana Bolena';`
```

Si ejecutamos este query utilizando una librería de bajo nivel para JS/TS, el resultado podría tener esta forma.

``` typescript
[
  {
    aa_id: 18,
    aa_customer: 'Ana Bolena',
    aa_status: 'Pending',
    aa_date: null,
    aa_requiredApprovals: 4,
    aa_agencyId: 7,
    aa_creditAssessmentId: 1,
    ag_id: 7,
    ag_code: '077',
    ag_name: 'Villa Revol',
    ag_address: 'Olimpia 2821',
    ag_area: 82,
    ca_id: 1,
    ca_customer: 'Ana Bolena',
    ca_creditLimit: '85000.00'
  }
]
```

Usando TypeORM, podemos expresar la misma query de esta forma
``` typescript
const application = await accountApplicationRepository.findOneOrFail(
    { customer: 'Ana Bolena' }, { relations: ["agency", "creditAssessment"] }
);
```

obteniendo un resultado con esta forma
``` typescript
AccountApplication {
  id: 18,
  customer: 'Ana Bolena',
  status: 'Pending',
  date: null,
  requiredApprovals: 4,
  agency: Agency {
    id: 7,
    code: '077',
    name: 'Villa Revol',
    address: 'Olimpia 2821',
    area: 82
  },
  creditAssessment: CreditAssessment {
    id: 1,
    customer: 'Ana Bolena',
    creditLimit: '85000.00'
  }
}
```

El ejemplo muestra que el uso de TypeORM permite que nos olvidemos mayormente de la sintaxis de SQL, aunque _sí tenemos que tener en cuenta el **modelo** relacional_, porque hay que describir explícitamente las relaciones que queremos incluir en la búsqueda, que se corresponden (aproximadamente) con los `JOIN` en una consulta SQL. 

---
**Nota** sobre diseño de ORM.  

Se podría pensar en una operación mediante ORM en la que no fuera necesario indicar, en cada query, qué relaciones se quiere incluir. El modelo de datos incluye las relaciones, en este caso, TypeORM "ya sabe" que una `AccountApplication` está relacionada con una `Agency` y un `CreditAssessment`, con lo cual podría incluir estas relaciones en forma automática.  
La desventaja de este automatismo es que en un modelo complejo con muchas relaciones, y en particular con _cadenas largas_ de relaciones (p.ej. una `AccountApplication` está relacionada con una `Agency`, que está relacionada con una `Person` -p.ej. el gerente-, que está relacionado con varios `Address` y así siguiendo), una query que en el programa parece sencilla, puede implicar una consulta compleja en la BD, la transmisión de una gran cantidad de información al backend, y el procesamiento de toda esta información para generar los objetos que corresponda, con la penalidad de performance.  

Estos problemas realmente aparecen al usar ORM, y se han propuesto distintas estrategias para resolverlos o mitigarlos. La solución que propone ORM mitiga los problemas de performance, con el costo de tener que ser consciente del modelo relacional al definir cada query. Personalmente, creo que es un buen compromiso.

---


## Algunas consideraciones adicionales

TypeORM también ofrece variantes para los casos en que nos convenga estar "más cerca" de la sintaxis de SQL. P.ej. la misma query se puede describir en TypeORM de esta forma.
``` typescript
const query = await accountApplicationRepository.createQueryBuilder('aa');
query.where('aa.customer = :customer', { customer });
query.leftJoinAndSelect('aa.agency', 'ag');
query.leftJoinAndSelect('aa.creditAssessment', 'ca');
const applications = await query.getOne();
```
Es interesante destacar que el _resultado_ de esta query sí es un objeto del modelo que se describe mediante decorators.

Otro ejemplo de este reflejo más directo de SQL en las operaciones descriptas mediante TypeORM son los operadores de búsqueda, p.ej. 
``` typescript
const application = await accountApplicationRepository.findOneOrFail(
    { customer: Like('% Bolena') }, { relations: ["agency", "creditAssessment"] }
);
```
En general, los operadores de las búsquedas en TypeORM se corresponden bastante directamente con los que pueden aparecer en una consulta SQL.

Finalmente, mencionemos que TypeORM incluye algunas características que van más allá de la relación directa con una BD relacional. 
Mencionamos: una variante más potente del concepto de cascada que incluye operaciones de alta, el soporte para cache en los resultados de una búsqueda, y el soporte para migraciones.
