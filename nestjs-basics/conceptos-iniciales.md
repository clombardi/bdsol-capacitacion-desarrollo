---
layout: default
---

## Conceptos iniciales - controller, provider, módulo
NestJS basa el desarrollo de una aplicación en tres conceptos: controller, provider y módulo.  

**Controllers**  
Son el punto de entrada de los request handlers. Un controller define un conjunto de rutas que va a atender, recibe los requests, y decide cómo generar la respuesta (y/o realizar la acción) que corresponde en cada caso.  
En realidad los controllers no deberían resolver, por sí mismos, la funcionalidad asociada a cada request; lo correcto es delegar esta tarea en los ... .

**Providers**  
Un _provider_ es cualquier objeto que (justamente) _provee_ la funcionalidad que necesitan los controllers para atender requests.  
Todas las operaciones relevantes (p.ej. consulta a BD, interacción con servicios externos) deben ser resueltas por los providers.

**Módulos**  
Son las unidades de organización de una aplicación NestJS. Cada módulo incorpora uno o varios controllers, y uno o varios providers. Una aplicación se construye como una red de módulos, que parte de un _módulo inicial_.  

Los tres conceptos se implementan como clases TS[^1] con mucho uso de decorators.

La documentación de NestJS recomienda, en la [página inicial](https://docs.nestjs.com/first-steps), tener cada módulo (incluyendo controllers y providers) en una carpeta separada.


## Cómo se arma una app
Receta en varios pasos

### Paso 1 - relación entre módulos
Los módulos pueden relacionarse entre ellos: un módulo puede _importar_ a otros módulos, y también definir que _exporta_ algunos de sus controllers o providers, que son los que van a poder usar los módulos que lo importen a él.
Si tenemos
``` typescript
@Module({
    controllers: [ToppingController],
    providers: [OnionService, TomatoService],
    exports: [OnionService]
})
export class ToppingModule { }

@Module({
    controllers: [PizzaController],
    providers: [PizzaService],
    imports: [ToppingModule]
})
export class PizzaModule { }
```
entonces dentro de `PizzaController` y `PizzaService` se puede usar `OnionService`, que es el componente que `ToppingService` decide exportar. 
Desde el `ToppingController` se pueden usar tanto `OnionService` como `TomatoService`, porque forman parte del mismo módulo.


### Paso 2 - módulo inicial
Entre todos los módulos hay uno que es el inicial. ¿Cuál? el que se hace referencia en el _bootstrap_ de la app, ver [la primer página de la doc NestJS](https://docs.nestjs.com/first-steps).
Se lo suele llamar `AppModule`. 
``` typescript
@Module({
    imports: [PizzaModule, HamburguerModule, SodaModule]
})
export class AppModule { }
```
Este módulo es la raíz de la red de módulos[^2] que forman una aplicación NestJS.  La importación es _transitiva_, o sea que en este caso los controllers de `ToppingModule` también se van a incluir.


### Paso 3 - Inyección de dependencias.
Los providers tienen que tener el decorator `@Injectable`.
``` typescript
@Injectable()
export class PizzaService {
    getCurrentFlavors() {
        return ['Mozzarella', 'Peperoni', 'Onion', 'MincedMeat']
    }

    // ... rest of provider implementation ...
 }
```
Nest se encarga de crear las instancias que necesite de cada provider (también de cada controller y módulo, claro) e _inyectarlas_ donde se necesite.

Los controllers (y también los providers) pueden acceder a las instancias de providers que maneja NestJS definiéndolas como _parámetros en el constructor_.
``` typescript
@Controller('pizza')
export class PizzaController {
    constructor(
        private readonly service: PizzaService,
        private readonly onionService: OnionService
    ) { }

    @Get('/flavors')
    getCurrentFlavors() {
        return this.service.getCurrentFlavors()
    }

    // ... other handlers ...
}
```
Al crear una instancia de `PizzaController`, NestJS le va a _inyectar_ (en el constructor) instancias de los providers, que tengan el mismo _tipo_ del parámetro del constructor.

### Fin de la receta
Con esto tenemos levantados y enganchados todos los elementos que forman parte de una app NestJS.

## Notas
Lo que acabamos de contar es _la forma más directa/usual_ de configurar una app NestJS.

El framework también incluye otras formas de generar, y de inyectar, instancias de providers. Ver las secciones "Property-based injection"  y "Manual instantiation" en la [página sobre providers en la doc Nest](https://docs.nestjs.com/providers), y tal vez también en otros lugares de la doc.

También hay varias variantes para establecer relaciones entre módulos, ver en la [página sobre módulos en la doc Nest](https://docs.nestjs.com/modules) desde "Shared modules" hasta el final.


<br/>

-----

[^1]: creo que también se puede usar JS para desarrollar aplicaciones NestJS; no averigüé sobre esto.

[^2]: en general la red de módulos tiene la forma de un _árbol_. Pero NestJS admite que haya dos módulos donde cada uno importa al otro, ver [la página sobre referencias circulares en la doc de NestJS](https://docs.nestjs.com/fundamentals/circular-dependency).
