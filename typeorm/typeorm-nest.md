---
layout: default
---

# Usando TypeORM en un proyecto NestJS
NestJS provee soporte específico para TypeORM, con un manejo muy similar al de Mongoose. En forma súper-resumida
- hay que incorporar un módulo especial, el `TypeOrmModule`.
- se establece la conexión en el módulo inicial usando `TypeOrmModule.forRoot`. Los  parámetros y opciones de conexión se pasan por parámetro.
- los repositorios se consideran recursos que maneja el `TypeOrmModule`. Para usar un repositorio en un provider hay que: importar el `TypeOrmModule.forFeature` en el módulo, e inyectar el repositorio en el provider usando el decorator `@InjectRepository`.

Respecto de lo que trabajamos usando TypeORM desde un script, hay una diferencia respecto de cómo indicarle a TypeORM qué tipos de entidad debe considerar.

Lo que sigue es una descripción _un poco_ más detallada. Para ver todos los detalles, se puede acceder a [la página en la doc de NestJS](https://docs.nestjs.com/techniques/database). 

**¡¡OJO!!**  
hay una página que en el menú de la doc que [se llama "TypeORM"](https://docs.nestjs.com/recipes/sql-typeorm), está dentro de "Recipes".  
Esa página describe una forma **mucho** más laboriosa de hacer la integración con TypeORM, que no usa el soporte específico que provee Nest.
La página que sí usa este soporte, o sea la que linkeamos más arriba, aparece en el menú como "Database", está dentro de "Techniques".


## Packages necesarios
A los packages que se necesitan para tener andando TypeORM en un proyecto TS, que son los que incorporamos en el script que trabajamos hasta ahora, hay que sumarle `@nestjs/typeorm` que es el soporte de NestJS específico para TypeORM.

O sea, para interactuar con una BD Postgres desde un proyecto Nest, hay que ejecutar lo siguiente por línea de comandos.
```
npm install --save typeorm reflect-metadata pg @nestjs/typeorm
```
Si se usa MySQL o MariaDB en lugar, en lugar de `pg` va `mysql`.


## Tipos de entidad
Esto es igual a como lo hicimos en el script, en la definición de las entidades como clases con decorators de TypeORM, NestJS no tiene ninguna influencia.


## Conexión
Para conectarse a una base usando TypeORM, importamos el `TypeOrmModule` en el módulo principal, que por lo general es el que se llama `AppModule`. Se usa el método `forRoot`, al que se le pasa por parámetro un objeto con los parámetros y opciones de conexión ... con un par de diferencias. 

Veamos un ejemplo.
```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { AccountApplicationModule } from './account-application/account-application.module';
import { AccountApplication } from './entities/account-application.entity';
import { Agency } from './entities/agency.entity';
import { City } from './entities/city.entity';
import { CreditAssessment } from './entities/customer-assessment.entity';
import { OfficeParty } from './entities/office-party.entity';
import { SafeDepositBoxVault } from './entities/safe-deposit-box-vault';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      "type": "postgres",
      "host": "localhost",
      "port": 5432,
      "username": "postgres",
      "password": "notapassword",
      "database": "nano_bank_model",
      "synchronize": true,
      "logging": true,
      "entities": [AccountApplication, Agency, City, CreditAssessment, OfficeParty, SafeDepositBoxVault],
      "keepConnectionAlive": true   
    }), 
    AccountApplicationModule
  ],
})
export class AppModule {}
```

La principal diferencia está en cómo se definen las `entities`: en lugar de indicar la ubicación de los archivos fuente, se indican las clases que modelan los tipos de entidad, notar que se están importando las clases.  
Comparemos con la misma especificación, en las opciones de conexión del script.
```typescript
    entities: [
        "src/entity/**/*.ts"
    ]   
```
Una consecuencia es que respecto de los parámetros de conexión, no da ninguna ventaja agrupar estas clases en una carpeta separada, se pueden ubicar en la carpeta del módulo relacionado a cada entidad o donde cada equipo de desarrollo prefiera. Con una definición como la del script, hay que poner las clases que modelan entidades en `src/entity`.  
Otra consecuencia de este cambio en la configuración, es que una definición de este estilo no podría ir en un JSON. Aunque la documentación de NestJS habilita a usar el archivo `ormconfig.json`, el hecho de indicar rutas en ese archivo podría complicar la ejecución del servicio NestJS en algunos ambientes. Por eso parece ser más seguro indicar las opciones de conexión en el código del módulo principal. 

Otra diferencia es que el soporte de NestJS define algunas opciones adicionales a las de TypeORM. En particular, en el ejemplo se está usando `keepConnectionAlive`, que es necesario para que funcione el hot reload de NestJS, como se indica al final en la [página de la doc de NestJS sobre hot reload](https://docs.nestjs.com/recipes/hot-reload).


## Repositorios
Como dijimos al principio de esta página, los repositorios se consideran recursos que maneja el `TypeOrmModule`. Por lo tanto, para usarlos en un módulo nuestro, tenemos que importar ese módulo usando el método `forFeature`, indicando las clases correspondientes a los repositorios que queremos tener disponibles. Se ve más fácil en un ejemplo.

```typescript
@Module({
    imports: [TypeOrmModule.forFeature([AccountApplication])],
    controllers: [AccountApplicationController],
    providers: [AccountApplicationService]
})
export class AccountApplicationModule { }
```

En los providers, tenemos que inyectar los repositorios, marcándolos con el decorator especial `@InjectRepository`. En ese decorator indicamos cuál es el repositorio que queremos usar, pasando la clase correspondiente por parámetro. 
```typescript
@Injectable()
export class AccountApplicationService {
    constructor(
        @InjectRepository(AccountApplication) 
        private readonly applicationRepository: Repository<AccountApplication>
    ) {}

    getAllApplications(): Promise<AccountApplication[]> {
        return this.applicationRepository.find();
    }
}
```
Como se ve en el ejemplo, los repositorios que obtenemos mediante inyección de dependencias le llegan al provider "listos para usar", podemos aplicar las mismas operaciones que trabajamos en el script, o cualquiera que encontremos en la doc de TypeORM.

En la definición del parámetro del constructor hay una cosa curiosa: hay que poner _dos veces_ `AccountApplication`, en el parámetro del decorator `@InjectRepository`, y en el parámetro de tipo de `Repository` que es un generic.  
Esto tiene que ver con la separación entre información que usan Typescript y VSCode en la edición/compilación, y la información que usa NestJS cuando el servicio está corriendo, o sea en la ejecución. El parámetro del decorator se ve en la ejecución pero no en la compilación, la información de tipo al revés, se ve en la edición/compilación pero no en la ejecución.

Para ver por qué al editor/compilador se le complica acceder a la información que está en el parámetro del constructor, veamos esta definición alternativa que (aunque no tiene sentido) funciona perfectamente
```typescript
function getEntity() {
    return AccountApplication;
}

@Injectable()
export class AccountApplicationService {
    constructor(
        @InjectRepository(getEntity()) 
        private readonly applicationRepository: Repository<AccountApplication>
    ) {}

    getAllApplications(): Promise<AccountApplication[]> {
        return this.applicationRepository.find();
    }
}
```
Para saber cuál es la clase, el compilador debería ser capaz de _ejecutar_ la función `getEntity()` ... justamente lo que los editores o compiladores no saben hacer, ejecutar código. Por eso es que no se puede especificar `Repository<getEntity()>` como tipo del repositorio.


## Para practicar
Les dejo algunos ejercicios que pueden servir para mirar un poco la documentación de TypeORM, y usar en una app NestJS algunas de las cosas que vimos sobre el script.

### Modificación de entidades
Agregar dos endpoints `PATCH`, uno para cambiar una solicitud de una sucursal, y otro para sumarle uno a la cantidad de aprobaciones requeridas. En ambos casos, identificando a la solicitud por el id.

Recordemos que en el modelo de solicitudes en Mongoose, le pudimos agregar un `method` que implementa el incremento en la cantidad de aprobaciones requeridas. ¿Dónde podría ir ese método en los modelos que se definen para TypeORM?

### Análisis crediticio
Agregar un endpoint `POST /account-applications/:id/creditAssessment`, que agregue el análisis crediticio correspondiente a una solicitud de cuenta. El único dato en el body del request debería ser el límite de crédito calculado. Si la solicitud ya tiene un análisis asociado, salir con status code `400 - Bad Request`.  
De acuerdo a cómo estén configuradas las cascadas, esto se puede considerar a nivel entidades solamente una modificación de la solicitud, o un alta de análisis más la modificación de la solicitud para engancharle el análisis recién creado.

### Eliminación
Agregar un endpoint para eliminar una solicitud. Definir qué se hace con el análisis crediticio que pudiera tener asociado.

### Filtros sobre solicitudes 
Agregar al endpoint `GET /account-applications` dos posibles query params: uno por id de sucursal, otro por nombre de la provincia. 
Para el de sucursal, se puede usar una relación one-to-many de sucursales a solicitudes, y pedirle las solicitudes a la sucursal. Para el de provincia, va a ser necesario armar un query builder. No es necesario hacer subqueries, se puede manejar con joins solicitud - sucursal - ciudad, y estableciendo una condición sobre la provincia de la ciudad.
Decidir qué hacer si se indican ambos parámetros.
