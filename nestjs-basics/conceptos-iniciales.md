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





-----
[^1] creo que también usar JS para desarrollar aplicaciones NestJS; no averigüé sobre esto.