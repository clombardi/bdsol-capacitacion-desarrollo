# Test - empezamos
En esta parte de la capacitación, vamos a trabajar con **testeo automático**, orientado a microservicios desarrollados en NestJS.

Antes de pasar a la práctica, demos algunos datos de contexto.


## Para qué testear
Sobre esto se ha escrito muchísimo, no tiene sentido seguir aportando bits a este tema.

Quiero destacar dos aspectos, uno porque me gustaría que pudiéramos trabajarlo en estas sesiones, otro que va a aparecer más adelante.  

El primero:  
tener un buen nivel de test automático nos saca un poco de _miedo al refactor_, la sensación de "lo que anda no-se-toca" que algunas veces nos lleva a que nuestros diseños degraden y que nuestro código, con el paso del tiempo, se vaya pareciendo cada vez más a lo que se conoce por el nombre en inglés [big ball of mud](http://www.laputan.org/mud/).  
Por eso, en estas actividades vamos a intentar hacer un par de ejercicios de refactor, y ver cómo que los tests sigan dando verde nos hace ganar un poco de tranquilidad.

El segundo:  
a medida que el ciclo de trabajo en un proyecto va estabilizando y mejorando, se intenta ir agregando prácticas que eviten que los cambios en el código rompan funcionalidades que están operativas.  
Por eso, en algún paso del proceso de integración, se ejecutará una batería de tests, y si no da verde, la integración para. 
Un punto posible para hacer esto es antes de aceptar un Pull Request.  
Para que esto tenga sentido, hay que tener un buen nivel de test. Dentro de la organización de Banco del Sol, este es un motivador fuerte para incluir test dentro de la capacitación.


## Qué testear
En general, se puede testear a distintos _niveles_
- desde el nivel más bajo, una función auxiliar
- hasta el nivel más alto, que es la interface del componente que estemos desarrollando. Para un microservicio, va a ser su API. Para una aplicación Web o Mobile, serán sus pantallas. 

Vamos a apuntarle a tests de distintos niveles, e incluso a debatir un poco a qué nivel _conviene_ testear.


## Con qué herramientas
Hay _muchas_ librerías y herramientas para testeo automático en el mundo JS / TS.

Entre los que podríamos llamar "frameworks de test", vamos a usar [Jest](https://jestjs.io/).  
Lo que motiva la elección es que Jest tiene soporte directo de NestJS. 
Pero ademas, es muy popular, y es (en mi experiencia) cómodo de usar.

Para tests de alto nivel, que para nosotros van a ser tests de providers y de servicios, vamos a usar el soporte que provee NestJS al respecto, que pueden ver [en la doc](https://docs.nestjs.com/fundamentals/testing).

> **Nota**  
> No hay diferencias relevantes entre las versiones 6 y 7 de Nest respecto del soporte de test. Por eso, aunque en Banco del Sol se está usando la versión 6, el link es de la documentación de la versión 7.


## Mock
Para cerrar esta introducción rapidísima, pongámosle nombre a un concepto relevante para armar tests.

Pensemos en tests de alto nivel, por ejemplo de la API de un servicio que obtiene datos de una base Mongo y de otro servicio.  
Cada test debe ser _aislado_, o sea, no depender de nada que no se haya configurado dentro del mismo test.
En particular, un test no puede confiar que la base contenga cierta info, tampoco en que *no* contenga cierta info. Tampoco se puede confiar en que el servicio al que invocamos esté levantado.

La técnica más común para poder hacer tests de alto nivel es incluir implementaciones simuladas de los componentes o recursos requeridos.  
P.ej. determinar que mientras se está ejecutando un test, cuando se hace un `find` sobre una colección Mongo/Mongoose, en lugar de ir a la base, brinde una respuesta prearmada, p.ej. salida de un JSON que acompaña al test. Esto permite conocer qué datos va a obtener la consulta, y por lo tanto, cuál es la salida que debería tener p.ej. un request handler.

A esta técnica de lograr que un test utilice implementaciones simuladas de componentes se la llama _mock_.
Vale usar el verbo "mockear", en el ejemplo anterior, para testear un request handler, estamos mockeando Mongoose, o sea, reemplazando la implementación "de verdad" de Mongoose por una que devuelve valores fijos.

Existen librerías para mockear distintos componentes, p.ej. vamos a usar [MongoDB In-Memory Server](https://github.com/nodkz/mongodb-memory-server) para mockear MongoDB.

El soporte de Nest para testing incluye mockeo de providers y módulos.


## Para arrancar
Hay que verificar que tengamos instalado en el proyecto el package de soporte de testing de Nest. Para eso miramos si en el `package.json` está
```
@nestjs/testing
```
en la lista de `devDependencies`.
Si no está, pues lo instalamos.

```
npm install @nestjs/testing@6.10
```
(el `6.10` del final es para que nos instale la librería en la versión 6, que es la misma en que tenemos Nest).