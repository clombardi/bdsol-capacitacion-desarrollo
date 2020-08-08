---
layout: default
---

# Una descripción de REST
En esta página, desarrollaremos las ideas principales de lo que se conoce como [REST](https://restfulapi.net/), una sigla de "Representational State Transfer" ... nombre que al menos a mí no me dice mucho.  
Como (¿casi?) toda sigla popular en informática, existen múltiples definiciones y descripciones de qué se entiende por REST.  
Aquí elegiremos una perspectiva que, esperamos, proporcione un buen marco para definir las interfaces de nuestros componentes.


## Contexto - sistemas altamente distribuidos
Para darle sentido a estas ideas, conviene empezar dando un poco de contexto.

En la bruma de los tiempos informáticos, una aplicación se alojaba en un único equipo de cómputo. Para aplicaciones grandes, se destinaban equipos con altas prestaciones, de aquí viene el concepto de _mainframe_. 
El mainframe puede concentrar todo lo necesario: las bases de datos que se requieran para almacenar información, los programas que procesan esa información, los componentes de interface que permiten la interacción con usuarios, que utilizan _terminales_ controladas por el mismo equipo central.

A medida que avanzan los tiempos, las tecnologías, el volumen de información que hay que manejar, y los requerimientos, se va desarrollando una tendencia a **distribuir** las responsabilidades de una aplicación en distintos equipos de cómputo.
Tal vez, uno de los primeros pasos fue la separación de las bases de datos en equipos específicos. 

La popularización de Internet, y en particular del protocolo HTTP, significa un empuje enorme a la idea de distribución. 
Al poder ejecutar código en un browser, se simplifica la separación entre frontend y backend.  
A su vez, la aplicación de las ideas, primero de _granja de servidores_, y luego de _computación en la nube_, multiplicaron las opciones para distribuir el backend en distintos equipos.
De la mano de estas ideas, surgen las _arquitecturas de microservicios_, que proponen la separación de la funcionalidad del backend en pequeñas unidades fuertemente independientes, que pueden instalarse en distintos equipos.

Esta tendencia hace de la _comunicación_ entre los distintos componentes en los que se distribuye el comportamiento de una aplicación, un aspecto trascendente en el desarrollo.


## Objetivos
En este contexto surge REST, un conjunto de principios que se recomiendan como guía para la organización de sistemas distribuidos, en particular los que usan tecnologías Web.

Esta guía está motivada por varios _objetivos_, haremos una breve mención de algunos de ellos.

**Escalabilidad**:  
se busca poder adaptar fácilmente una aplicación a un mayor nivel de volumen de información, cantidad y/o simultaneidad de pedidos. Una estrategia es agregar más equipos de cómputo.  
Por lo tanto, se buscan formas de comunicación que funcionen adecuadamente en escenarios con una gran cantidad de equipos que colaboran para sostener una aplicación. A este respecto, resulta importante que se puedan incorporar componentes que implementen estrategias de _balanceo de carga_ en forma transparente.

**Eficiencia**:  
se busca mejorar la eficiencia en la comunicación entre componentes. Además de la escalabilidad, otro factor que puede ayudar a la eficiencia es la inclusión de _caches_ de información de alto volumen de consulta y baja tasa de modificación.

**Neutralidad respecto de las tecnologías**:  
se favorecen las formas de interacción que no impongan restricciones sobre lenguajes de programación o herramientas utilizadas.

**Simplicidad**:  
se busca definir protocolos sencillos, que no aumenten la complejidad de los componentes de una aplicación.

**Modificabilidad**:  
se busca que los componentes puedan adaptarse a cambios, sin necesidad de detener el funcionamiento de la aplicación.

> **Nota**  
> Estos principios también están detrás de la concepción de las _arquitecturas basadas en microservicios_.


## Principios
A partir de estos objetivos (o _propiedades_ buscadas en una aplicación distribuida) se definen los varios principios que debe(ría)n guiar el diseño de los componentes y sus interfaces.


**Arquitectura cliente-servidor**  
Se debe distinguir claramente entre los componentes que proveen información -los _servidores_-, y los componentes que la consumen -los _clientes_-.  
Los clientes y los servidores deben comunicarse, únicamente, por una interface bien definida, mediante la cual los clientes realizan _pedidos_ que son atendidos por los servidores.

Esta definición de arquitectura lleva a _desacoplar_, o sea separar, el mantenimiento de la información de dominio (que manejan los servidores) de la interacción con los usuarios (que es responsabilidad de los clientes).  
En aplicaciones con interfaz Web, los clientes se ejecutarán en los navegadores, y se comunicarán con los servidores mediante pedidos en protocolo HTTP.


**Comunicación stateless**  
Cada pedido debe incluir toda la información que necesita un servidor para poder atenderlo. La respuesta o reacción ante un pedido, no puede depender de información que el cliente le haya enviado al servidor previamente, o más en general, de conocimiento que el servidor tenga sobre el cliente por fuera del incluido en el pedido.  
Todo el _estado de sesión_, o sea, toda la información relacionada con la interaccón con un usuario, debe residir en el cliente.

En particular, esto implica que la comunicación entre cliente y servidor no puede tener rasgos _conversacionales_, en el sentido que el comportamiento del servidor ante un pedido hecho por un cliente, no puede depender de interacciones previas con el mismo cliente.


**Por niveles (layered)**  
En la comunicación entre cliente y servidor, el cliente no puede conocer detalles acerca del servidor que va a atender cada pedido que haga.  
Esto permite agregar componentes intermedios, como _load balancers_ o _proxies_, sin que se modifique la forma de comunicación.


**Cacheable**  
La respuesta a un pedido puede indicar que la información es cacheable. El cache se puede implementar en el cliente o en componentes intermedios, con el obvio impacto en la eficiencia.


**Interface uniforme**  
La forma de realizar un pedido debe definirse a partir de reglas sencillas, que definan un formato _uniforme_ para acceder a la información. 
La intención es simplificar la comunicación entre componentes.  
REST propone basar los identificadores de cada pedido que define una aplicación, en el concepto de **recurso**, como describimos a continuación.


Mencionamos la existencia de un sexto principio, llamado **code on demand**, que no describimos porque es opcional, y es poco probable que vaya a ser aplicado en el Banco del Sol en un futuro próximo. 
Les interesades pueden consultar los links que se incluyen al final de esta página.


## Basado en recursos
En la propuesta REST, la comunicación se basa en el concepto de _recurso_, o sea, en la información al que hace referencia cada pedido que se le hace a un servidor.  
Cualquier **entidad** de la que hay que representar información se considera un recurso, p.ej. si pensamos en los ejemplos con los que hemos trabajado, encontramos
- países.
- solicitudes de cuenta.
- gastos.
- usuarios.
- colecciones de cualquiera de estas entidades.

La identificación de un pedido debe basarse en el recurso al que se quiere acceder o afectar.  
P.ej. si queremos acceder a la colección de todas las solicitudes de cuentas en una comunicación que use el protocolo HTTP, la URL (relativa) será
```
/account-requests
```
Ahora, se pueden realizar distintas **acciones** sobre un mismo recurso. P.ej. otra acción posible sobre la colección de solicitudes es agregar un elemento, o sea, informar sobre la existencia de una nueva solicitud.  
La acción a realizar sobre el recurso se informa por separado. En el caso de HTTP, típicamente usando los [métodos (o verbos) HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods).  

Hablaremos más en profundidad sobre la definición de endpoints HTTP un poco más adelante. Aquí sólo aclararemos que _una_ solicitud de cuenta es un recurso distinto a la _colección_ de solicitudes, por lo tanto va a tener una URL distinta, que va a incluir su _identificador_. Cada recurso dentro de una colección debe tener un [identificador único](https://en.wikipedia.org/wiki/Universally_unique_identifier).  
La URL para la solicitud cuyo identificador es `5f177780a4178205a49fb308` será. 
```
/account-requests/5f177780a4178205a49fb308
```

Los **procesos** que se quiera poder controlar mediante la interface de un servidor, también pueden ser considerados recursos. Por ejemplo: una generación masiva de resúmenes de cuenta, un envío masivo de mails, pueden ser considerados como recursos. P.ej. para lanzar una generación masiva de resúmenes de cuenta, se podría definir un `POST` sobre la URL
```
/account-statement-generation
```
y para consultar el estado de un proceso con identificador `5e929cfbfc8c3d6370dc0c1e`, podria hacerse mediante un `GET` a la URL
```
/account-statement-generation/5e929cfbfc8c3d6370dc0c1e
```


### Representación
La "R" en "REST" no se refiere a "recurso" sino a "representación".
La respuesta a un pedido de información es una _representación_ de un recurso, tal vez entre varias posibles.
En particular, se expresa en un formato de intercambio de información determinado (JSON, XML, YAML, etc.) y puede incluir sólo parte de la información asociada al recurso.

Analizaremos un poco más adelante, el manejo de _múltiples representaciones para un mismo recurso_.


## Para seguir leyendo
El link elegido para vincular la sigla [REST](https://restfulapi.net/) da una visión avanzada, más allá de lo que (hasta donde entiendo) se busca implementar en Banco del Sol, pero que puede ser interesante para investigar a futuro.

La entrada en [Wikipedia en inglés](https://en.wikipedia.org/wiki/Representational_state_transfer) está bien escrita.

Entre mis bookmarks tengo [este sitio de referencia](https://www.restapitutorial.com/), que incluye un video del que me gustó la explicación.

En cualquiera de estos materiales se puede consultar cómo surgió la propuesta de REST, descripción que no repetimos aquí ... porque puede accederse fácilmente.
