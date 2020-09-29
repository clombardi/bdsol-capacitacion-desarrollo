---
layout: default
---

# Picadito de comentarios
Cerramos esta sección con algunos comentarios puntuales, que surgen a partir de la revisión del código de algunos servicios del banco (tal como estaban a fines de septiembre de 2020).

## Evitar situaciones que podrían pasar inadvertidas

Analicemos este método de un provider.
``` typescript
async savePersonDevice(id: string, payload: SavePersonDeviceDto): Promise<Device> {
    const person = await this.personRepository.findOneOrFail(
      { id: parseInt(id, 10) },
      {
        relations: ['profile'],
      }
    )

    const previousDevice = await this.deviceRepository.findOne({ uid: payload.uid })
    if (previousDevice) {
        previousDevice.offlineToken = payload.offlineToken

        return this.deviceRepository.save(previousDevice)
    }

    const device = new Device()
    device.uid = payload.uid
    device.ipAddress = payload.ipAddress
    device.model = payload.model
    device.operatingSystem = payload.operatingSystem
    device.name = payload.name
    device.offlineToken = payload.offlineToken
    device.profile = person.profile

    expireByKey(redisCacheKeys.peopleHubPersonId(id))

    return this.deviceRepository.save(device)
}
``` 

En una vista rápida, este método agrega un `Device`.  
Pero en realidad, **no siempre** lo agrega: si ya existe un `Device` con el `uid` que viene en el `payload`, entonces le cambia solamente el `offlineToken`, y no agrega un `Device` nuevo.
Para ver eso, hay que darse cuenta del `return` en el medio. 

En general, conviene evitar que en un código haya condiciones o flujos que sean difíciles de apreciar a simple vista.

Si el código no ayuda, una forma es poner un comentario.  
``` typescript
async savePersonDevice(id: string, payload: SavePersonDeviceDto): Promise<Device> {
    const person = await this.personRepository.findOneOrFail(
      { id: parseInt(id, 10) },
      {
        relations: ['profile'],
      }
    )

    /*
     * If a device with the passed uid already exists, then just the offlineToken
     * must be changed. 
     * In this case, no new Device is added, cfr. the `return` inside the conditional.
     */
    const previousDevice = await this.deviceRepository.findOne({ uid: payload.uid })
    if (previousDevice) {
        previousDevice.offlineToken = payload.offlineToken

        return this.deviceRepository.save(previousDevice)
    }

    /*
     * If the previous condition is not met, then a new Device must be added.
     */
    const device = new Device()
    device.uid = payload.uid
    device.ipAddress = payload.ipAddress
    device.model = payload.model
    device.operatingSystem = payload.operatingSystem
    device.name = payload.name
    device.offlineToken = payload.offlineToken
    device.profile = person.profile

    expireByKey(redisCacheKeys.peopleHubPersonId(id))

    return this.deviceRepository.save(device)
}
``` 

Una alternativa es mover el código que crea el `Device` a un `else`, para que quede claro que es una cosa o la otra.
``` typescript
async savePersonDevice(id: string, payload: SavePersonDeviceDto): Promise<Device> {
    const person = await this.personRepository.findOneOrFail(
      { id: parseInt(id, 10) },
      {
        relations: ['profile'],
      }
    )

    const previousDevice = await this.deviceRepository.findOne({ uid: payload.uid })
    if (previousDevice) {
        /*
        * If a device with the passed uid already exists, then just the offlineToken
        * must be changed. 
        * In this case, no new Device is added, cfr. the `return` inside the conditional.
        */
        previousDevice.offlineToken = payload.offlineToken

        return this.deviceRepository.save(previousDevice)
    } else {
        /*
        * If the previous condition is not met, then a new Device must be added.
        */
        const device = new Device()
        device.uid = payload.uid
        device.ipAddress = payload.ipAddress
        device.model = payload.model
        device.operatingSystem = payload.operatingSystem
        device.name = payload.name
        device.offlineToken = payload.offlineToken
        device.profile = person.profile

        expireByKey(redisCacheKeys.peopleHubPersonId(id))

        return this.deviceRepository.save(device)
    }
}
``` 

Finalmente, notamos que al usar alguna de las alternativas para [traspaso masivo de datos desde el payload](./datos-del-payload), al quedar un código más compacto, se hace más fácil verlo. Veamos cómo queda, usando también el `else` y un comentario más compacto.

``` typescript
async savePersonDevice(id: string, payload: SavePersonDeviceDto): Promise<Device> {
    const person = await this.personRepository.findOneOrFail(
      { id: parseInt(id, 10) },
      {
        relations: ['profile'],
      }
    )

    const previousDevice = await this.deviceRepository.findOne({ uid: payload.uid })
    if (previousDevice) {
        // a device already exists - just change the offlineToken
        previousDevice.offlineToken = payload.offlineToken
        return this.deviceRepository.save(previousDevice)
    } else {
        // create a new device
        const device = new Device()
        copyProperties(device, payload, 
            ['uid', 'ipAddress', 'model', 'operatingSystem', 'name', 'offlineToken']
        );
        device.profile = person.profile

        expireByKey(redisCacheKeys.peopleHubPersonId(id))

        return this.deviceRepository.save(device)
    }
}
``` 


## Manejo de error - una certeza y algunas dudas
El siguiente es un método de controller.
``` typescript
@Delete(':personId/devices/:uid')
async deletePersonDevice(@Param('personId') personId: string, @Param('uid') uid: string): Promise<void> {
    try {
        return this.personsService.deletePersonDevice(personId, uid)
    } catch (error) {
        throw error
    }
}
``` 
En este método, la estructura `try - catch` está de más: si no se incluye y ocurre un error en el método del servicio, entonces el controller va a salir con el mismo error ... que es lo que está haciendo el `catch`.  
En general, atrapar una excepción y relanzar **la misma** excepción, sin hacer nada en el medio, es una operación superflua. 

En resumen, este método podría quedar así.
``` typescript
@Delete(':personId/devices/:uid')
async deletePersonDevice(@Param('personId') personId: string, @Param('uid') uid: string): Promise<void> {
    return this.personsService.deletePersonDevice(personId, uid)
}
``` 

Vamos ahora a este método de servicio
``` typescript
async updateFacephiTimeOutCount(personId: string): Promise<Person> {
    const person = await this.personRepository.findOneOrFail(
        { id: parseInt(personId, 10) },
        {
            relations: ['signUpRequest', 'contact'],
        }
    )

    person.signUpRequest.facephiTimeoutCount += 1

    try {
        return this.personRepository.save(person)
    } catch (err) {
        throw new Error(err)
    }
}
```
En este caso, no veo la ventaja de encerrar el error original dentro de un `new Error`. Tal vez la haya de acuerdo al `ExceptionFilter` standard del banco, pero lo dudo.  
Por otro lado, esto se hace si ocurre un error en el `save`, pero no si no encuentra el `personId`, notar que se está haciendo `findOneOrFail`, que si no encuentra ningún resultado, lanza un error. ¿Por qué se atrapa uno y no el otro?

Y un último comentario, respecto de la búsqueda de personas. En varios casos se usa el `findOneOrFail`, en varios otros, lo que ya trabajamos anteriormente de hacer un `findOne` y verificar si se encontró una persona. Va un ejemplo.
``` typescript
async saveAddress(id: number, payload: SaveCustomerAddressRequestDto): Promise<Address> {
    const person = await this.personRepository.findOne(
        { id },
        {
        relations: ['addresses'],
        }
    )
    const addresses = person.addresses

    if (isEmpty(person)) {
        throw new NotFoundException(ErrorCode.PERSON_NOT_FOUND)
    }
    /* ... más código ... */
}
```

Si no hay una razón para hacerlo a veces de una forma y a veces de otra, creo que conviene definir una forma standard de hacer la búsqueda de una persona, y respetarla. Sobre todo, por si llega alguien nuevo al equipo de desarrollo ¿de dónde copia?.

Entre paréntesis, en este código, por una cuestión de claridad, la asignación de `addresses` la pondría abajo de la validación de la `person` obtenida. Así se entiende: primero se busca la `person` (lo que incluye la validación), luego se procesa.

Finalmente, menciono otro caso
``` typescript
async getPersonDevices(personId: string): Promise<Device[]> {
    const person = await this.personRepository.findOneOrFail(
        { id: parseInt(personId, 10) },
        {
        relations: ['profile'],
        }
    )

    if (isEmpty(person)) {
        throw new NotFoundException(ErrorCode.PERSON_NOT_FOUND)
    }

    return this.deviceRepository.find({
        where: { profile: person.profile },
    })
}
```
Acá si no me equivoco, la validación nunca actúa, porque el `findOneOrFail`, si no encuentra, provoca que el método de servicio salga con excepción. 


## Último ejercicio
En `customer.service.ts`, hay dos métodos distintos que incluyen una sección de código _idéntica_, que tiene que ver con pasar a "no reales" todas las direcciones si se agrega o modifica una real. Armar un "método unificador" para este caso.  
Quien quiera, puede mirar un poco más fijo estos dos métodos, para ver cómo compactarlos / compartir más cosas.
