---
layout: default
---

# Funciones de búsqueda
En el servicio de reuniones tenemos varios métodos que buscan una reunión para luego hacer algo con ella. Este servicio está implementado sobre TypeORM.

A modo de ejemplo, mostramos el principio de dos de los métodos, de los que uno es el que usamos como ejemplo al hablar de validaciones.
``` typescript
async confirmMeeting(id: number): Promise<void> {
    const meeting = await this.meetingRepository.findOne({
        where: { id },
        relations: ['branch']
    });
    checkValidBranch(meeting.branch);
    /* ... el resto de la lógica ... */
}

async checkMeeting(id: number): Promise<boolean> {
    const meeting = await this.meetingRepository.findOne({
        where: { id },
        relations: ['branch', 'owner', 'attendees']
    });
    checkValidBranch(meeting.branch)
    /* ... el resto de la lógica ... */
}
```

En varios métodos se repite el mismo patrón: se busca una reunión por id, indicando algunas relaciones que se quieren obtener entre las que siempre está la sucursal, y se valida que la sucursal sea válida.

Otra oportunidad para aplicar DRY, con un detalle: las búsquedas no son _exactamente_ iguales, cambia la lista de relaciones, partiendo de que `branch` tiene que estar.  
Para resolver esto, vamos a definir un método adicional en el servicio que resuelva la búsqueda; la lista adicional de relaciones va a ser un parámetro de este método. 

En general, cuando las distintas secciones de código que queremos unificar tienen diferencias, tenemos que buscar la forma de contemplarlas. En este caso es sencillo, alcanza con un parámetro. 

``` typescript
async getMeeting(id: number, additionalRelations: string[] = []): Promise<Meeting> {
    const meeting = await this.meetingRepository.findOne({
        where: { id },
        relations: [...additionalRelations, 'branch']
    });
    checkValidBranch(meeting.branch);
    return meeting;
}

async confirmMeeting(id: number): Promise<void> {
    await this.fillFakeRepos();

    const meeting = await this.getMeeting(id);
    /* ... el resto de la lógica ... */
}

async checkMeeting(id: number): Promise<boolean> {
    await this.fillFakeRepos();

    const meeting = await this.getMeeting(id, ['owner', 'attendees']);
    /* ... el resto de la lógica ... */
}
```

Si necesitáramos utilizar la búsqueda de reuniones en _otro servicio_, la forma que respeta la organización de Nest consiste en 
- exportar el servicio donde esté el método `getMeeting` en el módulo de reuniones.
- importar el módulo de reuniones en los otros módulos donde se quiera usar la función.
- incorporar el servicio de reuniones como parámetro en constructor donde se necesite, para que Nest haga la inyección de la dependencia.


## Para practicar
En el servicio de personas de ejemplo, la búsqueda de una persona por id con la validación posterior de que existe, está 7 veces, en distintos servicios. Los encuentran buscando `isEmpty(person)`.  
Definir un método en el servicio que les parezca más adecuado para hacer la búsqueda + validación, y usarlo en el resto.  
En algún caso, además de la condición y las relaciones, se agregan más propiedades de búsqueda. Eso se puede contemplar con un parámetro adicional que se une (p.ej. usando spread) a los que define la función.

Una vez que esto anda, se puede pensar si no conviene que los `findOneOrFail` no conviene manejarlos también con este método, para que la forma de validar si una persona existe o no sea siempre la misma.


## Mini-extra - aceptando distintos tipos
En las búsquedas de persona por id, verán que a veces el id es un string, otras veces un número. En la función que armen, se puede definir un parámetro que admita cualquiera de las dos opciones, y las contemple en el código.  
En Typescript, al contrario que en Javascript, la función `parseInt` solamente acepta string. Se puede usar `Number(id)`, que acepta strings o números. Otra opción es usar `parseInt` _solamente si lo que llega es un `string`_, al respecto ver el siguiente código.
``` typescript
function powerfulParseInt(value: string | number): number {
    if (typeof value === "string") {
        return parseInt(value);      // acá value tiene tipo string
    } else {
        return value;                // acá value tiene tipo number
    }
}
```
Es interesante probar cómo se ven los diferentes tipos de `value` en el editor.

Más información sobre esto en [esta página en la doc de Typescript](https://www.typescriptlang.org/docs/handbook/advanced-types.html).

