## Operaciones en paralelo

Volvamos a este ejemplo
``` javascript
async function someOtherData() {
    try {
        const response = await axios.get("<other url>")
        const second_response = await doSomethingAsync(response.data)
        return /* other stuff */
    } catch (err) {
        /* error handling */
    }
}
```
en este caso, hay un **orden** en el que se tienen que evaluar las dos operaciones asincrónicas, porque la segunda (el `doSomethingAsync`) usa datos (el `response.data`) generados por la primera. 
La segunda operación tiene que _esperar_ los datos de la primera.

En otros casos, podría no ser así. P.ej. una operación podría necesitar hacer varias consultas independientes, y después trabajar con todos los resultados. P.ej. 
``` javascript
async function threeMix() {
    try {
        const response_1 = await axios.get("<url_1>")
        const response_2 = await axios.get("<url_2>")
        const response_3 = await axios.get("<url_3>")
        return doSomeMix(response_1.data, response_2.data, response_3.data)
    } catch (err) {
        /* error handling */
    }
}
```
donde ninguna de las tres URL dependen del resultado de las otras.  
Este código es correcto mirando la _funcionalidad_, pero se le puede hacer un cuestionamiento respecto de la _performance_.
De la forma en que está escrito, para lanzar la segunda consulta, va a esperar que llegue el resultado de la primera; lo mismo con la tercera respecto de la segunda. Las tres consultas se resuelven en forma **secuencial**. 
Muy probablemente, el tiempo de respuesta de `threeMix()` podría mejorarse si se pudieran lanzar las tres consultas en **paralelo**.  
Obviamente sacar los `await` no resuelve el problema: el `doSomeMix` _sí debe esperar_ que lleguen las 3 respuestas.

### Promise.all
Acá es donde las `Promise`s vienen al rescate.  
Existe el `Promise.all`, que recibe una _lista_ de promesas, y que devuelve una promesa cuya continuación (o sea, la función que se le pasa al `then`) recibe la lista de los resultados de cada promesa.  
Tal vez es más fácil verlo en código:
``` javascript
function threeMix() {
    return Promise.all([
        axios.get("<url_1>"), axios.get("<url_2>"), axios.get("<url_3>")
    ])
    .then(responses => doSomeMix(
        responses[0].data, responses[1].data, responses[2].data
    ))
    .catch(err => /* error handling */)
}
```
Si alguna de las operaciones falla, entonces el `Promise.all` falla y va al `catch`.  
Como siempre que manejamos promesas, importante no olvidarse el `return` al principio. 
Y por lo que "hablamos" en la página anterior, la función `threeMix()` puede usarse con `await` aunque no esté marcada con `async`, porque devuelve una promesa (que es la que devuelve el último `then`).


## Desafíos
Un par de desafíos para trabajar con esta técnica.

El primero: cambiar `threeMix` para que no diga tres veces `axios.get`, ni tenga que acceder explícitamente a cada elemento de `responses`.  
**Hint**: para lo segundo, usar la notación de "tres-puntos" para definir los argumentos de `doSomeMix`.

El segundo: 
cambiar `threeMix` para que _siempre_ llame a `doSomeMix`, aunque alguna/s de las llamadas HTTP falle. Para la/s que fallen, que en lugar de la response vaya `null`.