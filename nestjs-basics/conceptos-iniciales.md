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


### Cómo se arma una app
Receta en varios pasos

#### Paso 1 - relación entre módulos
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


#### Paso 2 - módulo inicial
Entre todos los módulos hay uno que es el inicial. ¿Cuál? el que se hace referencia en el _bootstrap_ de la app, ver [la primer página de la doc NestJS](https://docs.nestjs.com/first-steps).
Se lo suele llamar `AppModule`. 
``` typescript
@Module({
    imports: [PizzaModule, HamburguerModule, SodaModule]
})
export class AppModule { }
```
Este módulo es la raíz de la red de módulos[^2] que forman una aplicación NestJS.








<br/>

-----
[^1]: 
creo que también usar JS para desarrollar aplicaciones NestJS; no averigüé sobre esto.

[^2]:
en general la red de módulos tiene la forma de un árbol. Pero NestJS admite que haya dos módulos donde cada uno importa al otro, ver [la página sobre referencias circulares en la doc de NestJS](https://docs.nestjs.com/fundamentals/circular-dependency).