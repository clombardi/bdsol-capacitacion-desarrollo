---
layout: default
---

# Testear una aplicación Nest
Después del rápido repaso sobre conceptos básicos de test en general, y de Jest en particular, vamos a incorporar los elementos necesarios para testear, específicamente, un backend construido sobre NestJS.

Antes de arrancar con la parte técnica, tracemos un pequeño panorama que nos sirva de mapa de qué vamos a ir haciendo.  
Nos puede interesar hacer tests de componentes individuales de Nest, o bien (o también), hacer lo que se llaman _tests de integración_ o _end-to-end_. Veamos un poco cómo encarar cada posible tipo de test.


## Testeos de componentes individuales
Hay tres tipos de componentes que nos puede interesar testear _por separado_: controllers, providers, middleware.

**Controllers**:  
para testear específicamente el código de los controllers, podemos mockear los providers a los que llama el controller, e invocar a la interface del controller. De esta forma, el _único_ código "real" que se va a ejecutar en el test es el del controller: los servicios están mockeados, y los middleware no aplican porque se está llamando directamente al controller.  
Creo que testear a este nivel tiene sentido sólo para controllers que hacen algún procesamiento interesante. Para los que son "pasamanos" de un provider o poco más, puede ser más conveniente  armar tests centrados en los providers, accediéndolos mediante los controllers para que el testeo también "pase" por los mismos, como describimos a continuación.

**Providers**:  
para testear la lógica codeada en un provider, se pueden mockear las _fuentes de datos_ de ese provider. P.ej. Mongo/Mongoose, o los otros servicios a los que invoque.  
Se puede invocar, o bien directamente los métodos del provider, o bien un método de controller que sea poco más que un "pasamanos" al provider. La segunda estrategia nos puede servir para que un mismo test sirva ante cambios en la _interface_ de un servicio. Veremos un ejemplo de esto.

**Middleware**:  
para testear el código incluido en los elementos de middleware, se pueden definir controllers mock que tengan la configuración y/o comportamiento necesario para testear el middleware que nos interese. P.ej. que use un custom pipe implementado por nosotros, que lance una excepción que es manejada por un Exception Handler, etc..  
El testeo se puede hacer simulando una llamada a la API por HTTP, para esto va a servir la integración de [Supertest](https://github.com/visionmedia/supertest) incluida en el soporte de NestJS para testing.


## Testeos end-to-end
También podemos hacer **tests de integración**, conocidos como "end-to-end" o "e2e".  
En este caso, mockeamos lo que está más lejos de la API, o sea las fuentes de datos, e invocamos a la API. En estos tests probamos: el procesamiento inicial que hacen los providers, las transformaciones o integraciones que hacen los controllers, y las alteraciones o validaciones de los elementos de middleware.


## ¿Por dónde arrancamos?
Para implementar cualquiera de estos tipos de test, vamos a tener que familiarizarnos con [el soporte para testing de NestJS](https://docs.nestjs.com/fundamentals/testing), y también el mockeo de Mongo/Mongoose.

Los primeros tests que vamos a hacer van a comprobar _controllers_, porque creo que de esta forma el acercamiento a esta herramienta se hace un poco más paulatino, y podemos postergar un poco el trabajo con Mongo/Mongoose que agrega su parte de complejidad.
