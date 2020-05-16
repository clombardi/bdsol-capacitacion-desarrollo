## Procesamiento asincrónico - ejercicio integrador
Para practicar el uso de `Promise.all`, y ya que estamos algo de manejo de listas y estructuras, les proponemos este ejercicio.

Definir una función que recibe un código de país ISO-3, y devuelve la siguiente estructura
``` json
{
    "countryCode": "ARG",
    "countryName": "Argentina",
    "population": 43590400,
    "currency": { "code": "ARS", "name": "Argentine peso", "byUSD": 67.35 },
    "neighbors": [
        { "countryCode": "BRA", "countryName": "Brasil", "population": 206135893 },
        { "countryCode": "PRY", "countryName": "Paraguay", "population": 6854536 },
        { "countryCode": "URY",  "countryName": "Uruguay", "population": 3480222 },
        { "countryCode": "BOL",  "countryName": "Bolivia", "population": 10985059 },
        { "countryCode": "PRY", "countryName": "Chile", "population": 18191900 }
    ],
    "totalNeighborPopulation": 245647610,
    "covid19-last-record": {
        "date": "2020-05-14",
        "confirmed": 7134,
        "active": 4396,
        "recovered": 2385,
        "deaths": 353
    }
}
```

Usar las siguientes API públicas.

REST Countries, http://restcountries.eu/ .  
El endpoint `https://restcountries.eu/rest/v2/alpha/<codigo>` provee bastante de la data que se pide. Toma el código ISO-3. 

Free Currency Converter API, https://free.currencyconverterapi.com/ .  
Hay que obtener una API Key para usar gratis el servicio.  
El endpoint que nos va a servir es `https://free.currconv.com/api/v7/convert?q=USD_<currency_code>&compact=ultra&apiKey=<key>` .  
El `currency_code` es uno de los datos que se obtiene en el endpoint de REST Countries.

COVID19API, https://covid19api.com/ .  
El endpoint `https://api.covid19api.com/countries/<slug>` da la info sobre un país. Tomar los de la última fecha.  
Para obtener el slug hay que usar `https://api.covid19api.com/countries` y buscar por el ISO-2 code, que es uno de los datos que da el endpoint de REST Countries.

Con dos `Promise.all` debería alcanzar.

Obviamente, a partir de acá, que cada une le agregue/retoque lo que quiera.
