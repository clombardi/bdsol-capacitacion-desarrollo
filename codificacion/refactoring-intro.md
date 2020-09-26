---
layout: default
---

# Hablemos un poco sobre refactoring
En esta parte de la capacitación, vamos a volcar la mirada sobre **el código que escribimos**. No lo que hace, no las tecnologías y herramientas que usamos: el código puro y simple.  
En concreto, la propuesta va a ser trabajar con algunas situaciones en las que un  _refactor_ puede ayudar a que el código nos quede "mejor".

¿Qué es un _refactor_?  
Es cualquier acción de modificación sobre un conjunto de código, que no cambia la funcionalidad del código, o sea que hace lo mismo, con la misma eficiencia, el mismo grado de seguridad, etc..  

¿Por qué podría ser interesante hacer _refactors_?  
La respuesta a esta pregunta está relacionada a en qué aspectos un código puede "mejorar". Mencionemos algunos.
- para hacerlo más **legible**, o sea que una persona que no participó en la generación de ese código, entienda de qué se trata. 
- para que sea más **mantenible**, es decir, que si hay que hacer correcciones, cambios o agregados, sea sencillo entender dónde tocar, qué se puede aprovechar del código que está escrito; que no sea necesario tocar en muchos lados distintos para un cambio sencillo.
- para hacerlo más **expresivo**: que al (p.ej.) leer el código del servicio de créditos, entienda cuál es el modelo de créditos que se está manejando, cuáles son las operaciones relevantes, qué tipos de créditos existen, cuáles son las condiciones para otorgarlos, qué variantes existen para el cálculo de intereses, etc..
- para que nos quede más **lindo**, aumentar nuestro nivel de satisfacción con el producto de nuestro trabajo.


## Los refactors en el proceso de desarrollo
En un proyecto iterativo, en el que muchas funcionalidades van teniendo mejoras y agregados a medida que pasa el tiempo, se corre el riesgo de que el código nos quede una bola inmantenible, producto de las sucesivas capas de código que se van agregando.  
Este fenómeno es bien conocido en el ámbito de la programación, de hecho hay un [artículo de 1999](http://www.laputan.org/mud/mud.html) que describe la "gran pelota de barro" en el que se puede convertir el código de un proyecto.  
En este contexto, los refactors son útiles para organizar el código y dejarlo mejor preparado para los nuevos agregados.

Por otro lado, en un proyecto en el que siempre se está corriendo, se hace muy difícil encontrar tiempos para pensar e implementar refactors ... con la consecuencia de que el código se caotiza, y por lo tanto, cada vez se tarda más en hacerle cambios.  
Por eso es importante crear la cultura de asignar tiempo a operaciones de refactor, generar issues/tickets, planificarlos. 
Obvio que esto puede ser complicado, desde otros roles y miradas puede ser difícil entender por qué hay que dedicar tiempo a cambios en el código "que no se ven".
Ahí está nuestro lugar como profesionales, de defender la calidad de nuestro trabajo.

En un terreno menos político, al reorganizar el código, no es difícil que se escape algún bug. Por eso es importante mantener un _buen nivel de test_, que nos ayude a tener la confianza necesaria para cambiar lo que haga falta.

Finalmente, comentamos que resulta conveniente mantener activa la _actitud de buscar posibles mejoras_ al código, más allá de si hay tiempo para encararlas en ese momento o se anotan para hacerlas después. 
Personalmente, creo que esta actitud facilita a que cada desarrollador mantenga una cercanía con el código, o dicho de otra forma, evita que el código se convierta en un enemigo o una pesadilla. 
Ya que vamos a dedicarle a codear y a tareas anexas varias horas por día, mantengamos el gusto de hacerlo.


## Qué vamos a hacer en esta capacitación
Vamos a buscar distintas formas de aplicar el bien conocido principio DRY (Don't repeat yourself), o sea, que cada cosa que esté expresada en el código, esté una sola vez.

Como veremos, a veces es muy fácil descubrir oportunidades de refactor: el mismo código, o casi, aparece repetido, incluso en un mismo archivo.  
En otros casos, vamos a tener que usar el ingenio para darnos cuenta cómo podemos extraer una estructura común de distintas secciones de código de las que no es difícil darse cuenta que "hacen lo mismo", pero que tienen diferencias.

Espero que estos ejercicios ayuden a adquirir una visión del código más abarcativa, en lugar de pensar cada endpoint (o pantalla, o cualquier unidad de interfaz) por separado, pensar qué estoy modelando/describiendo, donde un mismo concepto puede cubrir varios endpoints, entonces no sería raro que exista código común que aplique a la resolución de todos ellos, o al menos de algunos.

