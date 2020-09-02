---
layout: default
---

## Decorators en TS - nano-introducción

Miremos la definición de un controller NestJS

``` typescript
import { Controller, Get, Post, Body } from '@nestjs/common';

@Controller('account-applications')
export class AccountApplicationController {
    constructor(private readonly service: AccountApplicationService) { }

    @Get()
    async getAccountApplications(): Promise<GetAccountApplicationsDto> {
        const applications = await this.service.getAccountApplications()
        return applications.map(application => { 
            return { ...application, date: application.date.format(dateStringFormat) } 
        });
    }

    @Post()
    async addAccountApplication(@Body() newApplicationData: AccountApplicationDto): Promise<AddResponseDto> {
        const newApplication: AccountApplication = {
            ...newApplicationData, 
            date: moment(newApplicationData.date, dateStringFormat), 
            status: newApplicationData.status as Status
        }
        const newId = await this.service.addAccountApplication(newApplication)
        return { id: newId }
    }
}
```

Las diferencias más notables de este código TS con uno JS son dos.  
Una es el tipado.  
La otra es el agregado de **decorators**{: style="color:Crimson"}, las indicaciones que empiezan con arroba `@`.  
Para quienes vienen de Java, tienen una sintaxis similar (si no igual) y el mismo propósito que las annotation de Java.
Piensen p.ej. en Spring, SpringBoot, Hibernate.

Los decorators son usados por librerías y frameworks para agregar configuraciones a medida que van procesando código TS.  
P.ej. ¿qué hace Nest cuando se encuentra con las anotaciones de un controller? Imaginemos
- `@Controller`: crea una instancia (o un pool, lo que le venga más cómodo) y la deja en algún registry donde la tenga a mano. 
  Además, asocia a esta instancia o pool con el string que va como parámetro, para armar las URL de los endpoints.
- `@Get` / `@Post`: agrega dos endpoints en el `express` sobre el que corre Nest. La URL la arma componiendo el string que tomó del `@Controller` con el que se pone en `@Get` o `@Post`. En cada endpoint, llama al método correspondiente en la instancia que se guardó en el registry.
- `@Body`: en la implementación del endpoint `post`, agarra el body y lo pasa como parámetro cuando llama al método de la instancia del controller.

Hasta donde entiendo, los decorators sólo se pueden poner en clases y elementos de una clase (atributos / constructor / métodos). Por eso todas las cosas que se tengan que decorar (p.ej. todos los elementos significativos de NestJS) se tienen que manejar como clases.  
Cada decorator se usa, al menos hasta donde vi, en un solo tipo de elemento, o sea es un decorator de clases, **o** de métodos, **o** de parámetros, etc. .


### Manejo de decorators - nos pasamos del lado de adentro de la librería
El manejo de decorators que hace un framework o librería no es magia china ni nada parecido, al menos en TS. Lo que viene es el _poquito_ que averigüé, justamente para que entendamos que es un concepto que podemos manejar.

En principio es muy sencillo: se define una _función_ para cada decorator, que recibe como parámetro el elemento (clase, método, parámetro, etc) que se está decorando.  

Vamos con un ejemplo: definamos el decorator de clases `@MakeItCool` que se usaría así:
``` typescript
class TrainStation {
    constructor(public name: string) {}
    description(): string {
        return `Station ${this.name}`
    }
}

@MakeItCool
class TrainLine {
    constructor(public end1: TrainStation, public end2: TrainStation) {}
    description(): string {
        return `${this.end1.description()} to ${this.end2.description()}`
    }
}
```
acá estamos aplicando el decorator a `TrainLine` pero no a `TrainStation`.

Para implementar el decorator, alcanza con definir una función `MakeItCool`, que recibe un parámetro.  
Qué es ese parámetro depende de para qué elemento es el decorator. En el caso de los decorators de clase, representa una representación de la clase ... de la que sé poco; creo que está relacionada con la implementación de clases de la que hablamos en la parte de object literals y clases en JS.

Para el caso no importa, lo único que va a hacer `MakeItCool` es llevar un registro de las clases que tienen el decorator.
``` typescript
const coolClasses = []

function MakeItCool(constructorFunction: Function) {
    coolClasses.push(constructorFunction)
}
```
Para probar, definir varias clases, algunas cool y otras no, y hacer un `console.log(coolClasses)` al final.

------
**Nota**{: style="color: SteelBlue"}:  
Ahora entendemos qué son los `import` de la primera línea en el ejemplo del controller NestJS: _son las funciones que implementan cada decorator_.

------


### Un chiche: parámetros en el decorator
Un pasito más: definamos un decorator que lleva un parámetro, p.ej. 
``` typescript
@ShowClassTo('Penny Lane')
class TrainStation {
    constructor(public name: string) {}
    description(): string {
        return `Station ${this.name}`
    }
}
```
El decorator simplemente manda por consola un mensaje que incluye al nombre de la clase y al valor del parámetro, en este caso `Behold the class TrainStation, Penny Lane`.

En este decorator tenemos que manejar datos de características distintas: por un lado está la clase que se está decorando (`TrainStation`), por otro el valor que se le pasa al decorator (`'Penny Lane'`).

Por lo (insisto, _poquito_) que vi, la estrategia de TS (al menos para los decorators de clase), es que la función que implementa el decorator devuelva _otra función_. Miren:
``` typescript
function ShowClassTo(whom: string) {
    return function (constructor: Function) {
        console.log(`Behold the class ${constructor.name}, ${whom}`)
    }
}
```
la función que implementa el decorator recibe los datos que se le pasan, y la función _que devuelve_ recibe la clase.


### Desafío
Definir un decorator de método, que permita marcar un método como "importante", y obtener un reporte de los métodos importantes en cada clase, o al menos la cantidad de métodos importantes en cada clase.  
Este decorator nos tiene que permitir marcar métodos de esta forma
``` typescript
class TrainLine {
    constructor(public end1: TrainStation, public end2: TrainStation) { }
    description(): string {
        return `${this.end1.description()} to ${this.end2.description()}`
    }

    @MarkAsImportantMethod
    isTrivial(): boolean {
        return this.end1 === this.end2
    }

    ends(): TrainStation[] {
        return [this.end1, this.end2]
    }

    @MarkAsImportantMethod
    isZeth(): boolean {
        return this.ends().some(end => end.name[0].toUpperCase() === 'Z')
    }
}
```
Si pido un reporte de los métodos importantes, tiene que indicar que la clase `TrainLine` tiene dos métodos importantes. Mejor si indica sus nombres.  
A mí me salió así:
```
important methods:
Map {
  'TrainLine' => [ 'isTrivial', 'isZeth' ]
}
```
(usé el `Map` que vimos al hablar de Generics).

Esto nos lleva a mirar y entender un poco de documentación sobre TS.


### Conclusión
Esto es sólo para tener una _mínima idea_ de qué se tratan los decorators.  
Creo que es sano sacarle un poco de la característica de magia negra a cada cosa que usamos. Con paciencia y sabiendo cómo leer y a quién preguntarle, todo pasa del lado de lo comprensible.
