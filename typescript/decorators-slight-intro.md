## Decorators en TS - nano-introducción

Miremos la definición de un controller NestJs

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

Los decorators son usados por librerías y frameworks para agregar configuraciones a medida que se van procesando un archivo TS.  
P.ej. ¿qué hace Nest cuando se encuentra con las anotaciones de un controller? Imaginemos
- `@Controller`: crea una instancia y la deja en algún registry donde la tenga a mano. Una instancia o un pool, lo que le venga más cómodo.  
  También, le asocia el string que va como parámetro, para armar las URL de los endpoint.
- `@Get` / `@Post`: agrega dos endpoints en el `express` sobre el que corre Nest. La URL la arma componiendo el string que tomó del @Controller con el que se pone en @Get o @Post. En cada endpoint, llama al método correspondiente en la instancia que se guardó en el registry.
- `@Body`: en la configuración del post, agarra el body y lo pasa como parámetro cuando llama al método de la instancia del controller.

Hasta donde entiendo, los decorators sólo se pueden poner en clases y elementos de una clase (atributos / constructor / métodos). Por eso todas las cosas que se tengan que decorar (p.ej. todos los elementos significativos de NestJS) se tienen que manejar como clases.  
Cada decorator se usa, al menos hasta donde vi, en un solo tipo de elemento, o sea es un decorator de clases, o de métodos, o de parámetros, etc..


### Manejo de decorators - nos pasamos del lado de la librería
El uso de decorators que hace un framework o librería no es magia china ni nada parecido, al menos en TS. Lo que viene es un _poquito_ que averigüé, justamente para que entendamos que es un concepto que podemos manejar.

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

Ahora entendemos qué son los `import` de la primera línea: son las funciones que implementan cada decorator.

### Un chiche: parámetros en el decorator

... tenemos dos niveles de parámetro: el valor que se le pasa al decorator, y el elemento decorado ...  
TS lo resuelve así ...

### Desafío

Definir un decorator de método, que cuente cuántos métodos se decoran en cada clase. 

### Conclusión

Esto es sólo para tener una mínima idea de qué se tratan los decorators. 
Creo que es sano sacarle un poco de la característica de magia negra a cada cosa que usamos. Con paciencia y sabiendo cómo leer y a quién preguntarle, todo pasa del lado de lo comprensible.