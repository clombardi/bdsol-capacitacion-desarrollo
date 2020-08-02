# Test de providers - mockeando Mongo
En secciones anteriores, definimos tests que apuntan a la lógica implementada en _controllers_ y _middleware_.  
En esta página, vamos a plantear tests que se focalicen en el código de los **providers**.

En particular, vamos a desarrollar tests para un provider que accede a un BD Mongo mediante Mongoose. 
Para lograr tests aislados (o sea, que no dependan del contenido que tenga una BD) y rápidos (o sea, que no tengan que crear una BD cada vez que se ejecutan), vamos a utilizar un _mock de MongoDB_.

En concreto, vamos a incorporar el package [MongoDB In-Memory Server](https://github.com/nodkz/mongodb-memory-server), que permite definir una base Mongo que reside en memoria, y que por lo tanto es volátil (además de muy rápida).  
Esta estrategia permite que en los tests se verifique, no solamente la lógica que se aplica a partir del resultado de un query, sino también la forma en que se define el query, y también las definiciones de esquemas de Mongoose.  
Destacamos que se está ejecutando el mismo código de Mongoose que en producción, solamente reorientándolo a una base en memoria, que tiene la misma interface que una base MongoDB "normal".


## El provider a testear
Para este test, vamos a cambiar de dominio: vamos a trabajar con un módulo sobre solicitudes de cuentas (`AccountRequest`).  
Este módulo se apoya en una colección de Mongo, a la que se accede mediante Mongoose. Se utiliza el soporte para Mongoose que provee Nest, descripto en una [sección anterior](../mongoose-nest/mongoose-en-nest.md). El dominio es muy similar al que utilizamos en esa sección.

### El modelo de datos
Esta es la definición del esquema Mongoose para las solicitudes de datos.
``` typescript
export const AccountRequestSchema = new mongoose.Schema({
    customer: { type: String, required: true },
    status: { type: String, enum: Object.values(Status) },
    date: Number,
    requiredApprovals: { type: Number, default: 3 }
})

AccountRequestSchema.virtual('isDecided').get(
    function(): boolean { return ['Accepted', 'Rejected'].includes(this.status) }
);

AccountRequestSchema.method({
    hasDate: function(): boolean { return !!(this.date && this.date != 0) },
    month: function(): number | undefined {
        return this.hasDate() ? moment(this.date).utc().month() + 1 : undefined 
    },
})
```
Observamos que hay datos que residen en la base, y otros calculados mediante definiciones agregadas en el esquema.

### El provider
El provider define dos métodos. 
Uno permite acceder a las solicitudes registradas, pudiendo filtrar por cliente y/o status.
El otro permite agregar una solicitud.
``` typescript
@Injectable()
export class AccountRequestService {
    constructor(@InjectModel('AccountRequest') private accountRequestModel: Model<AccountRequestMongoose>) {}

    async getAccountRequests(conditions: AccountRequestFilterConditions): Promise<AccountRequest[]> {
        // implementación
    }

    async addAccountRequest(req: AccountRequestProposal): Promise<string> {
        // implementación, devuelve el id del nuevo AccountRequest
    }
}
```

Se utilizan estas interfaces.
``` typescript
export interface AccountRequestProposal {
    customer: string,
    status: Status,
    date: moment.Moment, 
    requiredApprovals: number
}

export interface AccountRequest extends AccountRequestProposal {
    id: string,
    month: number, 
    isDecided: boolean
}

export interface AccountRequestFilterConditions {
    customer?: string,
    status?: string
}
```
Una `AccountRequestProposal` incluye los datos necesarios para registrar una nueva solicitud, una `AccountRequest` incluye todos los datos asociados a una solicitud.

En ambos casos, el provider hace las transformaciones necesarias entre "sus" interfaces, y el formato definido en el esquema Mongoose.


### El controller
El controller define dos endpoints, que se corresponden con las dos operaciones del provider.
``` typescript
@Controller('account-requests')
export class AccountRequestController {
    constructor(private readonly service: AccountRequestService) { }

    @Get()
    async getAccountRequests(@Query() conditions: AccountRequestFilterConditions): Promise<AccountRequestDTO[]> {
        // implementacion
    }

    @Post()
    async addAccountApplication(@Body() newRequestData: AccountRequestProposalDTO): Promise<AddResponseDTO> {
        // implementacion
    }
}
```
donde se utilizan estas interfaces.

``` typescript
export interface AccountRequestProposalDTO {
    customer: string,
    status: string,
    date: string,
    requiredApprovals: number
}

export interface AccountRequestDTO extends AccountRequestProposalDTO {
    id: string,
    month: number,
    isDecided: boolean
}

export interface AddResponseDTO {
    id: string
}
```

La única responsabilidad propia del controller es realizar las transformaciones entre las interfaces DTO y las que maneja el provider.


## Estrategia - mock de la base
Los tests apuntan a verificar el código incluido en los métodos del provider. 
Este código tiene dos responsabilidades: acceder a la base mediante Mongoose, y realizar las transformaciones de datos de acuerdo a su interface.

Como lo indicamos al principio, los tests que definamos van a activar una base MongoDB que va a trabajar en memoria, utilizando el package [MongoDB In-Memory Server](https://github.com/nodkz/mongodb-memory-server).  
Para utilizar este package, se le solicita que cree una base Mongo en memoria; esta base soporta _las mismas operaciones_ que una base Mongo "real" de modificaciones y búsquedas sobre una colección.  
Una vez creada la "base virtual", puede obtenerse una URI para acceder a la misma.
``` typescript
import { MongoMemoryServer } from 'mongodb-memory-server';

const mongoServer = new MongoMemoryServer();
const memoryMongoUri = await mongoServer.getConnectionString();
```
Esta URI puede ser utilizada para configurar Mongoose. Si utilizamos Mongoose mediante el soporte de NestJS, podemos configurar el `MongooseModule` de esta forma.
``` typescript
MongooseModule.forRoot(
    memoryMongoUri, { useNewUrlParser: true, useUnifiedTopology: true }
)
```
Destacamos (nuevamente) que se está utilizando la versión **operativa** de Mongoose, que ejecutará operaciones sobre la base Mongo de la que se indica la URI en la configuración. La única diferencia de la operación en los tests, es que la base Mongo a la que accede Mongoose es "virtual". Pero de esto, Mongoose ni se entera.  
El código del provider, y eventualmente del controller, también son los operativos.

Los siguientes esquemas muestran los elementos involucrados en un entorno operativo y en la ejecución de test. 
![elementos en entorno operativo](./images/structure-in-operations.jpg)

![elementos en entorno de test](./images/structure-in-test.jpg)

Se observa que el único que cambia es la BD Mongo. Esta estrategia maximiza el código operativo que interviene en los tests.



> **Nota**  
> En la documentación del package `mongodb-memory-server`, se aclara que realiza una instalación **completa** de MongoDB en `./node_modules`.  
> Si se va a utilizar `mongodb-memory-server` en varios proyectos, tal vez convenga instalarlo de forma tal que realice una única instalación "propia" de MongoDB. Para esto, hay que instalar una variante de `mongodb-memory-server`, p.ej. `mongodb-memory-server-core`.  
Los detalles se pueden consultar en la [doc del package](https://github.com/nodkz/mongodb-memory-server).


### Una alternativa - mockear Mongoose
