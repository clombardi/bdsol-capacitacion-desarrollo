---
layout: default
---

# Agrupando acciones comunes
Si hicimos todo lo indicado en la [página anterior](./un-test-de-controller-nest), entonces vamos a tener tres tests de controller ... y en los tres vamos a tener _el mismo_ código de inicialización del `TestingModule`.

``` typescript
describe('Country data service', () => {
    it('test 1', async () => {
        // codigo para inicializar testModule y theController
        // codigo para verificar
    });

    it('test 2', async () => {
        // codigo para inicializar testModule y theController
        // codigo para verificar
    });

    it('test 3', async () => {
        // codigo para inicializar testModule y theController
        // codigo para verificar
    });
});
``` 
Donde para los tres tests, el "código para inicializar testModule y theController" va a ser _todo_ esto
``` typescript
const testModule = await Test.createTestingModule({
    controllers: [CountryDataController],
    providers: [CountryDataService],
})
    .overrideProvider(CountryDataService)
    .useValue(fakeCountryDataService)
    .compile();
const theController = testModule.get(CountryDataController);
``` 
_igual_ en los tres. El "código para verificar" sí va a ser particular de cada test.

Para evitar esto, Jest provee una forma de definir código que se ejecuta _antes de cada test_ en una suite. Para esto, se agrega un bloque `beforeEach` dentro de la suite. El bloque `beforeEach` no lleva nombre, solamente se le pasa el código que hay que ejecutar antes de cada test.  
En nuestro ejemplo, podemos armar la suite de esta forma.
``` typescript
describe('Country data service', () => {
    beforeEach(async () => {
        const testModule = await Test.createTestingModule({
            controllers: [CountryDataController],
            providers: [CountryDataService],
        })
            .overrideProvider(CountryDataService)
            .useValue(fakeCountryDataService)
            .compile();
        const theController = testModule.get(CountryDataController);
    })

    it('test 1', async () => {
        // codigo para verificar, va a utilzar el controller generado
        await theController.getCountryData("SHR");
    });

    it('test 2', async () => {
        // codigo para verificar
    });

    it('test 3', async () => {
        // codigo para verificar
    });
});
``` 
El código incluido en el test 1 subraya que los tests tienen que poder utilizar `testModule` y `theController`, que están definidos en el bloque `beforeEach`.  
En la forma en que armamos el test, esto _no ocurre_, porque las constantes _son locales_ a la función del `beforeEach`.
Por suerte, se pueden definir variables a nivel de suite. Como queremos asignarle valor dentro del `beforeEach`, no pueden ser constantes. 

Queda así.
``` typescript
describe('Country data service', () => {
    let testModule: TestingModule;
    let theController: CountryDataController;

    beforeEach(async () => {
        testModule = await Test.createTestingModule({
            controllers: [CountryDataController],
            providers: [CountryDataService],
        })
            .overrideProvider(CountryDataService)
            .useValue(fakeCountryDataService)
            .compile();
        theController = testModule.get(CountryDataController);
    })

    it('test 1', async () => {
        // codigo para verificar, va a utilzar el controller generado
        await theController.getCountryData("SHR");
    });

    it('test 2', async () => {
        // codigo para verificar
    });

    it('test 3', async () => {
        // codigo para verificar
    });
});
``` 
Ahora sí, como son variables definidas en la función _de la suite_, son accesibles por todos los bloques internos, el del `beforeEach` (que les da valor), y los de cada test (que las usan).


## beforeEach y beforeAll
Jest también incluye un "primo" de `beforeEach`, llamado `beforeAll`. La diferencia es que `beforeEach` se evalúa una vez _antes de cada test_, mientras que `beforeAll` se ejecuta _una sola vez_, antes de empezar con la suite.

El detalle de `beforeEach`, `beforeAll`, y algunas otras indicaciones, se puede ver en la [página sobre inicialización y finalización de tests de la doc de Jest](https://jestjs.io/docs/en/setup-teardown).

Pensar si en este caso se puede reemplazar `beforeEach` por `beforeAll`. 
- Si sí se puede, pensar cómo sería un caso en que _no_ se pueda reemplazar `beforeEach` por `beforeAll`.
- Si no se puede, pensar cómo sería un caso en que sea indistinto usar `beforeEach` o `beforeAll`.
