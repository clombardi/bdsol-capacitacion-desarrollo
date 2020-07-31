# Un test de controller NestJS
Vamos a volver al dominio de la información sobre países, con el que trabajamos hace un tiempo. 

El módulo que vamos a testear, define estas interfaces.
``` typescript
export interface CountryData {
    countryCode: string
    countryNames: { es: string, en: string, br: string }
    population: number
    currency: { code: string, name: string, symbol: string }
    neighborCountryCodes: string[]
    continentCode?: string
}

export interface CountryDataDTO {
    countryCode: string
    countryName: string
    population: number
    currency: { code: string, name: string, symbol: string }
    continent?: string
}
``` 

El controller tiene un único método `@Get` que llama a un provider para que le brinde un `CountryData`. A su vez, el controller devuelve un `CountryDataDTO`. 

Aún sin ver nada de código, sólo mirando las dos interfaces, podemos entender qué transformaciones hace el controller.
- "elimina" el atributo `neighborCountryCodes`.
- de los `countryNames`, se queda con uno, o los combina de alguna forma, para generar el `countryName`.
- transforma el código de continente en ¿el nombre?

El análisis del código del controller nos permite confirmar estas sospechas.
``` typescript
const continents = {
    "AF": "África",
    "AM": "América",
    "AS": "Asia",
    "EU": "Europa",
    "OC": "Oceanía"
}

@Controller('countries')
export class CountryDataController {

    constructor(private readonly countryDataService: CountryDataService) { }

    @Get(':countryCode')
    async getCountryData(@Param("countryCode") countryCode: string): Promise<CountryDataDTO> {
        if (countryCode === 'ZZZ') {
            throw new NastyCountryException("I just don't like the country ZZZ")
        }
        const serviceData = await this.countryDataService.getCountryData(countryCode);
        const finalData: CountryDataDTO = {
            ..._.pick(serviceData, ["countryCode", "population", "currency"]),
            countryName: serviceData.countryNames.es
        }
        if (serviceData.continentCode) { 
            const continentName: string | undefined = continents[serviceData.continentCode];
            if (continentName) { finalData.continent = continentName }
        }
        return finalData;
    }
}
``` 

OK, lo que dijimos, más una excepción para un código específico de país.

El objetivo es armar tests en los que se pruebe el código que estamos viendo. 
En estos tests, lo _único_ que nos va a interesar del `CountryDataService` es su _interface_. En principio es esta.
``` typescript
export const SERVICE_NAME = 'CountryData cool service';

export class CountryDataService {
    async getCountryData(countryCode: string): Promise<CountryData> {
        // implementación que no nos interesa
    }
}
``` 
Lo de "en principio" es porque en realidad también puede interesarnos **qué excepciones** puede lanzar el provider. En este caso, son dos.

Si el `countryCode` es `TPT`, entonces lanza esta excepción  
`throw new ImATeapotException('Teapot country not supported')`.

Si no encuentra información para el código de país que se suministra, lanza una excepción de esta forma  
``` typescript
throw new NotFoundException({
    message: `Country ${countryCode} unknown`,
    originalError: /* un error */,
    serviceName: SERVICE_NAME
})
```


## Uso del controller con un provider mockeado