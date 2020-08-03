# Variaciones y reflexiones
Con (todo) el material que estudiamos, creo que tenemos una buena base para encarar tests que verifiquen un backend Nest.

Cierro con algunos comentarios particulares.


## Test de integración
Todos los tests que armamos apuntan a verificar el funcionamiento de un componente específico: un controller, un provider, un componente de middleware. Esto se asocia a la idea de _test unitario_.

Un tipo de test distinto, que también tiene sentido, es el _test de integración_. En estos tests, se accede a la API de lo que se quiere testear (en nuestro caso, microservicios que forman un backend), y se verifica que se obtienen los resultados esperados, utilizando el máximo posible del código operativo (o sea, mockeando lo menos posible).

Juntando elementos que presentamos en los [tests de middleware](./test-de-middleware.md) y los [test de providers](./test-de-providers.md), podemos armar tests de integración:
- se testea accediendo a la API como en los tests de middleware, y
- se mockea la base de datos agregando datos de prueba, como en los tests de providers.


## Un caso de refactor - para qué sirven los tests
Supongamos que se descubre que el uso de virtuals y methods de Mongoose es inconveniente, p.ej. por razones de performance.  
¿Qué hacer con el mes y el `isDecided` en las solicitudes? Los puede calcular el servicio.

Este es un ejemplo de **refactor**: un cambio en el código que no modifica su funcionalidad.  
En rigor, el planteado es un refactor chiquito, puede haber otros más ambiciosos.

¿Y cómo sé que no se me escapa nada al hacer un refactor? _Porque tenemos los tests_.  
Les propongo hacer este pequeño cambio, y correr los tests para verificar que nada dejó de andar.

Obviamente, en refactors más potentes, también puede ser necesario cambiar algunos tests, p.ej. si paso responsabilidades de controller a provider o viceversa, es probable que haya que ajustar los tests de los componentes que hacen más que antes, o que hacen menos que antes.  
Pero en todo caso, los _tests de integración_ me dan un primer indicio de que "vamos bien".


## Variantes de la ejecución de tests
Jest tiene varias [opciones de ejecución](https://jestjs.io/docs/en/cli). Hay dos que merecen definiciones separadas en los `package.json` de los proyectos NestJS.
```
    "test:watch": "jest --watch",
    "test:cov": "jest --coverage",
```
Describimos brevemente estas dos opciones.

La opción `--watch` deja levantado el Node sobre el que se ejecutan los tests, y cada vez que cambiamos el código, los ejecuta de nuevo. Como para ver "en vivo" si voy rompiendo algo a medida que modifico el código.

La opción `--coverage` brinda una tabla con información de _cobertura de test_ (o sea, _test coverage_). Es una indicación de cuánto del código se está usando en los tests. 

Finalmente, les comento otra opción: `--verbose`, esto muestra la lista de tests en cada suite aunque se ejecuten muchas suites.