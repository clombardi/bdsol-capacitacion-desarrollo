## Combinando datos de distintas fuentes
OK, tenemos andando un endpoint que da datos sobre un país. Ahora queremos agregarle datos sobre COVID, que la respuesta quede así:
``` javascript
{
    code: 'ARG',
    name: 'Argentina', 
    population: 43590400,
    internetDomain: '.ar',
    covid19LastRecord: {
        "date": "2020-05-14",
        "confirmed": 7134,
        "active": 4396,
        "recovered": 2385,
        "deaths": 353
    }
}
```
Para resolver este endpoint, necesitamos datos de dos fuentes distintas: la que ya veníamos usando de datos generales de un país, y una para los datos específicos de COVID.

Podemos resolver todas las consultas en un mismo provider ... o también podemos **definir un provider separado** para que resuelva las consultas sobre COVID.  
La primera opción es fácil: se organiza el código en la clase del provider y listo.  
En esta página vamos a explorar alternativas para la segunda. 

Antes que nada: ¿_por qué_ nos interesaría definir un provider separado, qué ventaja nos daría?  
Además de distribuir responsabilidades entre distintos componentes, podrían aparecer otras consultas en los que se use la info sobre COVID, pero no los datos generales. En estos casos, sería cómodo incorporar solamente el provider de data COVID, sin necesitar traerme toda la funcionalidad de datos de países.  
Esto obviamente es una decisión que puede depender fuertemente del escenario, acá solamente se plantean alternativas y tips para la resolución según qué opción se elija.

Entonces, definimos que vamos a tener un provider con esta forma 
``` typescript
@Injectable()
export class CovidDataService {
    async getLastRecord(countryCode: string): Promise<CovidRecord> {
        // ... implementation ...
    }
``` 
y una interface asociada `CovidRecord`.

A partir de esa decisión, surgen varias cuestiones a resolver
1. ¿en qué _módulo_ poner el `CovidDataService`?
1. ¿en qué nivel se integra la info de los dos providers?
1. qué pasa si el `CovidDataService` no da respuestas.
1. cómo manejar la identificación de país (el `countryCode`) si distintos providers usan distintos ids.

Vamos de a una.

### En qué módulo
Tenemos dos opciones: los dos providers van en el mismo módulo, o se crea un `CovidDataModule` separado.  
Qué criterio puede ayudar a decidir: la relevancia que tenga en mi aplicación la data COVID. Si es un conjunto de datos relevante, pues puede merecer un módulo separado. Si son datos accesorios a la información de un país, entonces pueden ser dos providers en un mismo módulo.


La implementación de cualquiera de las dos opciones es sencilla.  

Si **van en el mismo módulo**, simplemente se agrega el `CovidDataService` a la lista de `providers` del `CountryDataModule`:
``` typescript
@Module({
    controllers: [CountryDataController],
    providers: [CountryDataService, CovidDataService]
})
export class CountryDataModule { }
``` 

Si van en **módulos separados**, entonces hay que acordarse de dos cosas: exportar el provider en el módulo de COVID
``` typescript
@Module({
    providers: [CovidDataService],
    exports: [CovidDataService]
})
export class CovidDataModule { }
``` 
e importar el módulo de COVID en el de países.
``` typescript
@Module({
    controllers: [CountryDataController],
    providers: [CountryDataService]
    imports: [CovidDataModule]
})
export class CountryDataModule { }
``` 
observar que el módulo de COVID no declara ningún controller. Sirve solamente para proveer servicios a otros módulos.

### En dónde se integra la info
O sea, quién es el encargado de integrar la info que viene de los providers de info de país e info de COVID en la respuesta que otorga el controller.

Hay (al menos) tres opciones para esta integración: 
- la resuelve _el mismo provider_ de info de países que tenemos definido.
- la resuelve _un nuevo provider_ en el `CountryDataModule`.
- la resuelve _el controller_.

El componente que se encarga de la integración tiene que inyectarse al `CovidDataService`, agregando un parámetro en el constructor. P.ej. si es un nuevo provider tendría esta forma
``` typescript
@Injectable()
export class CountryIntegratedDataService {
    constructor(private readonly covidDataService: CovidDataService) { }

    // ... implementation ...
}
``` 

Donde sea que hagamos la integración, podemos usar `Promise.all` para acceder a las dos fuentes en paralelo.
``` typescript
const [countryData, covidData] = await Promise.all <CountryInfo, CovidRecord> ([
    this.countryDataService.getInfo(countryCode),
    this.covidDataService.getLastRecord(countryCode)
])
``` 
Notar que se puede especificar el tipo que va a tener cada elemento del array. Esto nos preserva el chequeo de tipos sobre los usos que le demos a `countryData` y `covidData`.

### Qué pasa si no hay datos COVID
Hay que contemplar el caso en que el servicio de info COVID no tenga disponibles datos para el pais.  
La **primera decisión a tomar** es si en este caso, _el controller de info integrada_:
- debe dar error, que puede ser un 404, o bien
- da la info sin incluir los datos de COVID.

Si elegimos la primera opción, podemos manejarnos como lo vimos al trabajar con el [manejo de errores](./manejo-de-errores.md). 

Supongamos que elegimos la segunda opción.  
¿Qué valor le damos a `covidData` si no hay data?. Nos gustaría que fuera `undefined` y después hacer algo así:
``` typescript
if (covidData) { 
    result.covid19LastRecord = covidData
}
``` 
Ahora, si el `covidDataService` sale con excepción, _tenemos que atraparla donde estamos integrando_. Pero la invocación a este servicio está dentro de un `Promise.all`, ¿qué hacemos?  
Van dos opciones.

Una es armar una función que resuelva el try-catch, se puede definir como método para poder acceder al `covidDataService` sin tener que pasarlo como parámetro
``` typescript
async getCovidLastRecord(countryCode: string): Promise<CovidRecord> {
    try {
        return await this.covidDataService.getLastRecord(countryCode)
    } catch (err) {
        return undefined
    }
}
``` 
(dejo como _mini-desafío_ implementar esta función _adentro_ del método donde se hace la integración).  
En el `Promise.all` se usa este nuevo método.
``` typescript
const [countryData, covidData] = await Promise.all <CountryInfo, CovidRecord> ([
    this.countryDataService.getInfo(countryCode),
    this.getCovidLastRecord(countryCode)
])
``` 

La otra alternativa para atrapar la excepción es usar el `catch`, sabiendo que finalmente lo que devuelve el método del `covidDataService` es una promesa
``` typescript
const [countryData, covidData] = await Promise.all <CountryInfo, CovidRecord> ([
    this.countryDataService.getInfo(countryCode),
    this.covidDataService.getLastRecord(countryCode).catch(() => undefined)
])
``` 
Fíjense que nos ahorramos la necesidad de definir una función adicional. Conviene tener siempre presente que detrás de los `async/await` hay promesas que se pueden manejar también usando `then/catch`, para poder usar la opción más práctica en cada lugar.

<br/>


------
**Nota**{: style="color: SteelBlue"}:  
Volviendo al debate sobre en dónde hacer la integración, veo que el manejo de errores es más bien una cuestión relacionada al endpoint que a un servicio que brinda datos.  
Esto es un argumento a favor de llevar la integración más cerca del controller, o sea, hacerla o bien en el controller mismo, o bien en un provider específico de integración.

------

<br/>

OK, ya manejamos la excepción para que no rompa el servicio de datos integrados.  
Tal vez aparezca **otro** problema, este de tipos: si en el `tsconfig.json` tenemos prendido el  `strictNullChecks`, o el `strict` que lo incluye, entonces va a decir que `undefined` no es un valor aceptado para algo que se declara como `CovidRecord`:
![No acepta `undefined` donde espera un CovidRecord](./images/undefined-not-admitted.jpg)

Para ser ultra-prolijos con los tipos, tenemos que declarar que aceptar `undefined` como resultado de la consulta al `covidDataService`.
``` typescript
const [countryData, covidData] = await Promise.all<CountryInfo, CovidRecord | undefined>([
    this.countryDataService.getInfo(countryCode),
    this.covidDataService.getLastRecord(countryCode).catch(() => undefined)
])
```
Si este fenómeno se repite en varios lados, puede ser útil darle un nombre a este tipo
``` typescript
type MaybeCovidRecord = CovidRecord | undefined
```
y después usarlo donde haga falta.

### Manejar distintos ids para el mismo objeto
Hasta acá nos manejamos con el mismo código para los servicios de información general de país y de información sobre COVID.  
Ahora, esto podría no ser así, en particular para consultas a servicios externos, en los que la representación de la información no es decisión nuestra. 

En el ejemplo que estamos trabajando, efectivamente, el servicio de información de país recibe el código ISO-3 (o sea de 3 letras, p.ej. "ARG") y el de información de COVID el código ISO-2 (p.ej. "AR").  
Como hicimos primero la consulta de información general, definimos que el código para _nuestros_ endpoints es el ISO-3. ¿Cómo obtener el código que espera el servicio de COVID?

Siguiendo con este ejemplo, el sitio externo de info de un país incluye en la respuesta el código ISO-2. Entonces podríamos obtener ese dato y usarlo para la consulta al servicio COVID. Pero esto nos rompería el Promise.all, ahora las consultas hay que hacerlas en forma secuencial
``` typescript
const countryData: CountryInfo = await this.countryDataService.getInfo(countryCode)
const covidData: (CovidRecord | undefined) = await this.covidDataService
        .getLastRecord(countryData.countryIso2Code)
        .catch(() => undefined)
```
Una alternativa sería establecer que se le envía el mismo código a todos los servicios, y el que lo tenga que transformar, que se arregle. Pero entonces ... el servicio de COVID **debería usar al `CountryDataService`** para obtener el código ISO-2.  
Esta variante parece _peor_ que la anterior: se estaría accediendo **dos veces** a la info de país. Pero ... considerando que los códigos ISO de los países no cambian básicamente nunca, si establecemos una _cache_ en el servicio COVID, la necesidad de una consulta extra se va a atenuar.

Este es un caso en el que nos viene bien una cache a nivel de provider. De esto vamos a hablar más adelante.

Por ahora cerramos indicando que si definimos dos módulos separados `CountryDataModule` y `CovidDataModule`, entonces cada uno de ellos tiene que importar al otro. O sea, una referencia circular. Siguiendo [la doc de Nest](https://docs.nestjs.com/fundamentals/circular-dependency), hacemos una _forward reference_  para sortear este problema.
``` typescript
@Module({
    controllers: [CountryDataController],
    providers: [CountryDataService],
    exports: [CountryDataService],
    imports: [forwardRef(() => CovidDataModule)]
})
export class CountryDataModule { }
```
y _lo mismo_ en `CovidDataModule`.


