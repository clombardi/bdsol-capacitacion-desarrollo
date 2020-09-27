---
layout: default
---

# Validaciones
Pensemos en un servicio que incluye varias operaciones relacionadas con sucursales, entre ellas, agregar una cuenta y confirmar una reunión. El servicio incluye (tal vez entre otros) un módulo de cuentas y otro de reuniones.  
En ambos casos, y eventualmente en otras operaciones relacionadas con sucursales, hay que validar que la sucursal esté activa. Si no, hay que salir con un status code `400 - Bad Request`.

En el servicio de cuentas encontramos este código.
``` typescript
async addCurrentAccount(payload: CurrentAccountDTO): Promise<void> {
    const branch = await this.branchRepository.findOne(
        { where: { branchNumber: payload.branchNumber } }
    )
    if (!branch.isEnabled) {
        throw new BadRequestException(ErrorCode.BRANCH_NOT_ENABLED);
    }
    /* ... el resto de la lógica ... */
}
```

Este es el código del servicio de reuniones.
``` typescript
async confirmMeeting(id: number): Promise<void> {
    const meeting = await this.meetingRepository.findOne({
        where: { id },
        relations: ['branch']
    });
    if (!meeting.branch.isEnabled) {
        throw new BadRequestException(ErrorCode.BRANCH_NOT_ENABLED);
    }
    /* ... el resto de la lógica ... */
}
```
Esta validación podría repetirse en otros lugares, en donde hay que garantizar coherencia en la excepción y el mensaje.

A su vez, si después se descubre que también hay que validar que la sucursal exista, hay que transformar el código de validación
``` typescript
if (isEmpty(branch)) {
    throw new NotFoundException(ErrorCode.BRANCH_NOT_FOUND);
} else if (!branch.isEnabled) {
    throw new BadRequestException(ErrorCode.BRANCH_NOT_ENABLED);
}
```
y esto hay que cambiarlo _en cada lugar_ donde se hace la validación de sucursal.


## Concentrar las validaciones en una función
¿Cómo podemos aplicar la idea de DRY en un caso así?   
Se puede definir una función que implementa las validaciones necesarias sobre sucursales.
``` typescript
export function checkValidBranch(branch: Branch) {
    if (isEmpty(branch)) {
        throw new NotFoundException(ErrorCode.BRANCH_NOT_FOUND);
    } else if (!branch.isEnabled) {
        throw new BadRequestException(ErrorCode.BRANCH_NOT_ENABLED);
    }
}
```

y simplemente la llamamos en cada lugar en donde se hace una validación, p.ej. 
``` typescript
async confirmMeeting(id: number): Promise<void> {
    const meeting = await this.meetingRepository.findOne({
        where: { id },
        relations: ['branch']
    });
    checkValidBranch(meeting.branch);
    /* ... el resto de la lógica ... */
}
```

Las funciones de validación pueden ir en un archivo único para todo el proyecto, o en un `xxx.validations.ts` en cada módulo. 
En cualquier caso, al definir las validaciones en archivos aparte, se simplifica saber qué cosas se están validando, y se va armando una biblioteca que se puede consultar para ver si me estoy salteando alguna validación que debería hacer.


## Para practicar
En el servicio de personas, hay varios accesos a una persona, y después de cada uno, se valida que la persona exista. Armar una función que implemente esta validación, y usarla en todos lados.

