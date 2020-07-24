# API REST - algunas particularidades
Después de haber planteado [criterios generales para definir una API HTTP que sea REST](./api-rest.md), vamos a concentrarnos en algunas cuestiones puntuales, para completar lo que vamos a decir sobre este tema.


## Métodos: PUT vs PATCH (vs POST)
Los métodos HTTP más usuales sobre un recurso son: GET / POST / PUT / PATCH / DELETE. _Cinco_ métodos.  
Por otro lado, la idea de "CRUD" define _cuatro_ operaciones: crear / modficar / borrar / obtener. 

Parecería que nos "sobra" un método HTTP. En particular, respecto de la _modificación_: tenemos `PUT` y `PATCH`.

En la [documentación en MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods), encontramos que
- el [`PUT`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/PUT) se define como _idempotente_, y habla de "reemplazar la representación del recurso con el payload" (o sea, el body).
- el [`PATCH`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/PATCH) se define como no idempotente, y habla de "instrucciones sobre cómo modificar un recurso".

**Por las dudas**:  
"Idempotente" quiere decir que el efecto de hacer una operación muchas veces seguidas, es el mismo de hacer la misma operación una vez sola.

A partir de estas indicaciones, las sugerencias son
- Usar `PUT` para hacer modificaciones _completas_ de un recurso, en donde en el body se incluyen los mismos datos que en el alta.
- Seguro, usar `PATCH` para modificaciones que no sean idempotentes. Por ejemplo: aumentar el precio de un producto (o varios) un 10%, donde lo que se envía en el body es el porcentaje de aumento.
- En principio, usar `PATCH` también para modificaciones _parciales_ de un recurso, en donde en el body se incluyen solamente los datos que se quiere modificar.

> **Nota**  
> En Internet se van a encontrar referencias que indican que para un alta, al menos en algunos casos resultaría "más correcto" usar `PUT` que `POST`.  
> Creo que es un debate demasiado "fino", que no aporta al menos para la construcción de microservicios de la forma en que se desarrollan y usan en el contexto de Banco del Sol.
> Por eso, aquí se mantiene la recomendación de usar `POST` para las altas, y `PUT` o `PATCH` para las modificaciones.


## Query parameters
En principio, los query parameters se utilizan (como su nombre lo indica) como parámetros de búsqueda. Por ejemplo, para buscar personas nacidas en 1986 que tengan una cuenta corriente, podemos definir un endpoint de este estilo:
```
GET /persons?birthYear=1986&hasCheckingAccount=true
```

> **Nota**  
> Para criterios o esquemas más complejos de búsqueda, se recomienda revisar propuestas existentes, de las en Internet se encuentran fácilmente, antes de inventar.

Hay otras cuestiones que también pueden especificarse mediante query params.

Una es el _criterio de ordenamiento_. P.ej.
```
GET /persons?sort_by=surname.desc
```
está solicitando que la lista de personas vuelva ordenada por apellido en forma descendente.

También pueden incluirse datos de  _paginación_. Una opción común es `offset` (o sea desde qué posición en base-0) + `limit` (cuántos resultados como máximo).
```
GET /persons?offset=60&limit=20
```

Obviamente, todos estos aspectos se pueden combinar
```
GET /persons?birthYear=1986&hasCheckingAccount=true&sort_by=surname.desc&offset=60&limit=20
```

Un resumen de estos usos de los query params se puede encontrar en [este post](https://www.moesif.com/blog/technical/api-design/REST-API-Design-Filtering-Sorting-and-Pagination/). En particular, se describen distintas opciones para paginación, y también para búsquedas un poco más sofisticadas.


## Headers
Mediante el uso adecuado de los [headers HTTP](https://developer.mozilla.org/es/docs/Web/HTTP/Headers), podemos aumentar las capacidades de nuestras API.
Además de los headers que están definidos, podemos agregar los que necesitemos para nuestra aplicación.

Al ser información que podemos incluir en cualquier request, y que está separada de la información de negocio, se utilizan para el manejo de aspectos no-funcionales / cuestiones arquitecturales / cross-cutting concerns, o como cada une les quiera llamar.  
Entre ellos destacamos _autenticación / autorización_, _tracing_ y _cache_. 
Vamos a tratar estos temas en una etapa siguiente de esta capacitación, al ampliar nuestra mirada de un microservicio al backend completo.

Veremos dos usos específicos de headers en lo que queda de esta página.


## Versionado
En una aplicación que evoluciona con el tiempo, es probable que surjan distintas _versiones_ de un mismo microservicio.  
A su vez, un costo de permitir la evolución separada de cada componente, es que resulta probable que deba haber distintas versiones del mismo servicio que estén _operativas_ en un mismo momento.  
Pasa un tiempo desde que se publica la versión 2 de un microservicio X, hasta que todos los componentes que lo utilizan publican, a su vez, nuevas versiones que utilizan la versión 2 de X. Hasta ese momento, deben estar operativas _a la vez_, las versiones 1 y 2 de X.

En tal caso, en cada pedido debe indicarse _qué versión_ del servicio debe atenderlo.  
Más en general, decimos que cualquier API puede tener versiones, y por lo tanto, al hacer un pedido a una API, debe indicarse cuál es la versión que corresponde utilizar.

Existen (al menos) dos propuestas para indicar la versión requerida de una API en un pedido.  
Una es indicar la versión _en la URL_, lo que da lugar a endpoints de esta forma
```
GET /v1/account-requests
```
La otra opción que mencionaremos es incluir un _header_ que indique la versión requerida del servicio, o sea.
```
GET /account-requests

Headers:
{ 
    version: '1.0',
    ...
}
```

En [este artículo](https://medium.com/hashmapinc/rest-good-practices-for-api-design-881439796dc9) se trata brevemente la cuestión del versionado.

## Distintas representaciones de un mismo recurso
Si la URL identifica a un recurso, y el único método para obtener información de `GET` ¿cómo podemos especificar que en distintos contextos, pueden requerirse distintas representaciones, o "vistas", de un mismo recurso?

Podemos tomar el ejemplo de la información sobre países con la que trabajamos en la etapa anterior. 
Pensemos en dos representaciones: una _abreviada_ que incluye sólo los datos básicos del país, y otra _extendida_, que incluye información sobre los países vecinos y sobre COVID.

También podríamos pensar en representaciones alternativas para una colección. P.ej. si queremos un _resumen_ que incluya ciertos datos estadísticos.


### Lo más rápido: distintos endpoints
Una opción sencilla es definir _distintos endpoints_.
```
GET /countries/:id/brief
GET /countries/:id/extended
```
Pero aquí estamos rompiendo el criterio de identificación por recurso, en realidad el _recurso_ al que nos estamos refiriendo es el mismo (el país), y sólo cambia la _representación_ que nos interesa en cada caso.


### Lo "más RESTful": header Accept y content negotiation
Una alternativa "más RESTful", aunque tal vez más engorrosa, sea usar el [header `Accept`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept) en el request, para indicar la representación deseada.  
Este header se utiliza más comúnmente para indicar preferencias de _formato_ de intercambio de información.  
Observando los requests que genera Postman, por defecto se incluye el siguiente header
```
Accept: '*/*'
```
A su vez, en las responses generadas por NestJS, se incluye el header `Content-Type`,  en el que se indica que el formato de la respuesta es JSON.
![headers de Postman y de Nest](./images/headers-postman-nestjs.jpg)

El cliente puede especificar en el header `Accept` qué formatos acepta, en orden de prioridad. P.ej. 
```
Accept: 'application/xml, application/json'
```
El proveedor debería considerar este orden de preferencias, y entregar la información de la mejor forma posible, considerando sus propias posibilidades de generar la respuesta. Esto es lo que se conoce como [content negotiation](https://developer.mozilla.org/en-US/docs/Web/HTTP/Content_negotiation), ver también la [definición de HTTP](https://tools.ietf.org/html/rfc7231#section-3.4).

En la forma en que venimos trabajando, ante un `Accept` con este valor, se debe entregar la representación en JSON ... que es la única que sabemos generar.

Ante un valor de `Accept` que no incluya JSON como posibilidad, p.ej. 
```
Accept: 'application/xml'
```
se debería responder con un status code `406 - Not Acceptable`, como se describe [en la página MDN de este status code](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/406).  
Hasta donde entiendo, NestJS no incluye una funcionalidad que controle el valor de `Accept`, deberíamos hacerlo en el código. Una alternativa sería definir un [Guard](../nestjs-basics/guards.md) que rechace los pedidos `GET` que no acepten un JSON como respuesta.

**Yendo a lo nuesto**  
El header `Accept` también puede incluir indicaciones específicas para el proveedor, conocidas como "vendor types" o "vendor MIME types". Estos tipos empiezan con `application/vnd.`, y se acumulan con el formato usando `+`.  
Se puede usar estas indicaciones para que el cliente especifique cuál de las distintas representaciones posibles de un recurso espera como respuesta.  
Por ejemplo, para indicar que se espera la representación abreviada, este sería un posible valor del header `Accept`.
``` 
Accept: application/vnd.bdsol.country.extended+json 
``` 
Aquí estamos indicando que esperamos la versión extendida de la información de un país, en formato JSON. 

Para procesar estos valores de header, hay que analizar el valor del header en el request handler. 
``` typescript
@Get(':countryCode')
async getCountryData(@Headers("accept") acceptValue: string, Param("countryCode") countryCode: string): Promise<any> {
    if (acceptValue === 'application/vnd.bdsol.country.extended+json') {
        return this.getCountryExtendedData(countryCode);
    } else if (acceptValue === 'application/vnd.bdsol.country.brief+json') {
        return this.getCountryBriefData(countryCode);
    } else {
        throw new NotAcceptableException(`Cannot honor accept value ${acceptValue}`);
    }
}
```
Si se quiere hacer un parseo del valor del header `accept`, se puede usar el package `accepts`, que da un poquito (poquito) de asistencia para esta tarea.

Encontré una explicación sobre este tema (después de buscar bastante) en [este post](https://softwareengineering.stackexchange.com/questions/390956/are-different-endpoints-to-display-the-same-resource-in-different-ways-restful).



### Otra alternativa: usar query params
Mencionemos finalmente una tercer opción, que es indicar la representación deseada como un query param.
```
GET /countries/:id?representation=brief
GET /countries/:id?representation=extended
```

Con un formato similar, podríamos indicar explícitamente  _qué atributos_ se quiere obtener sobre un recurso.  
```
GET /countries/:id?fields=countryCode,countryName,population,totalNeighborPopulation
```


## Agrupar información de distintos recursos en un mismo request
Otra cuestión ligada a qué información requiere un cliente, es que existen situaciones en las que se necesita información de varios recursos relacionados, mientras que REST sugiere que se incluya, en cada endpoint, información de un único recurso. 

Se pueden reconocer (al menos) dos aspectos que pueden considerarse problemáticos.
1. el mayor tiempo de reacción del frontend por la necesidad de esperar varias respuestas del backend.
1. la mayor complejidad en el cliente por tener que hacer varios requests, tal vez relacionados entre sí.

Una estrategia para abordar esta problemática es el establecimiento de _caches_ a distintos niveles, esto apunta en partiular al primer aspecto.

Una solución que abarca ambos aspectos es la definición de una capa específica de servicios dedicados a un frontend específico, los llamados BFF o "back-for-front".


