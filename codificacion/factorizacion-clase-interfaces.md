---
layout: default
---

# Factorizando "clase-interfaces"
Ya sabemos que Typescript no permite incluir decorators en las interfaces, y por lo tanto, cualquier estructura que podría ser una interface pero que quiero anotar, debe ser una clase. 
Un ejemplo típico son los _DTO_ que representan los payload de requests y responses.

En varias situaciones, nos encontramos con estructuras que tienen varias properties en común. P.ej., nos toca construir un controller para dar de alta distintos tipos de cuenta.

``` typescript
@Controller('/accounts')
export class AccountsController {
    @Post('/current')
    addCurrentAccount(@Body() payload: CurrentAccountDTO): Promise<void> {
        /* ... implementacion ... */
    }

    @Post('/savings')
    addSavingsAccount(@Body() payload: SavingsAccountDTO): Promise<void> {
        /* ... implementacion ... */
    }

    @Post('/international')
    addInternationalAccount(@Body() payload: InternationalAccountDTO): Promise<void> {
        /* ... implementacion ... */
    }
}
```

Como no es difícil de imaginar, las estructuras que describen los distintos tipos de cuenta, son bastante parecidas.

``` typescript
export class CurrentAccountDTO {
    @IsString() @IsNotEmpty()
    readonly accountNumber: string

    @IsInt() @Min(1) @Max(99999)
    readonly branchNumber: number

    @IsString() @Length(22, 22) @IsNotEmpty()
    readonly cbu: string

    @IsNumber() @Min(0) @IsOptional()
    readonly allowedOverdraft?: number
}

export class SavingsAccountDTO {
    @IsString() @IsNotEmpty()
    readonly accountNumber: string
 
    @IsInt() @Min(1) @Max(99999)
    readonly branchNumber: number

    @IsString() @Length(22, 22) @IsNotEmpty()
    readonly cbu: string

    @IsNumber() @Min(0) @IsDefined()
    readonly interestRate: number

    @IsNumber() @Min(1) @Max(31) @IsOptional()
    readonly interestApplicationDay?: number
}

export class InternationalAccountDTO {
    @IsString() @IsNotEmpty()
    readonly accountNumber: string
 
    @IsInt() @Min(1) @Max(99999)
    readonly branchNumber: number

    @IsString() @Length(22, 22) @IsNotEmpty()
    readonly cbu: string

    @IsString() @Length(5, 39) @Matches(/^[A-Z]{2}[0-9]{2}.*$/) @IsOptional()
    readonly iban?: string

    @IsString() @Length(3,3) @IsOptional()
    readonly alternateCurrency?: string
}
```

Estas definiciones tienen que ser `class`, porque incluyen las decoraciones de `class-validator`. Podrían tener además otras, p.ej. las de Swagger.

Los tres tipos de cuenta incluyen las `accountNumber`, `branchNumber`, y `cbu`. Cada tipo incluye sus properties particulares.  
Este es un ejemplo excelente para buscar cómo aplicar DRY: nuestro código está diciendo tres veces que las cuentas tienen número de cuenta, número de sucursal y CBU; quedaría mejor si esta información estuviera una sola vez. 

Para estos casos, podemos aprovechar que cada DTO está definido como una clase, y usar _herencia de clases_. Nos queda así.

``` typescript
class AccountDTO {
    @IsString() @IsNotEmpty()
    readonly accountNumber: string

    @IsInt() @Min(1) @Max(99999)
    readonly branchNumber: number

    @IsString() @Length(22, 22) @IsNotEmpty()
    readonly cbu: string
}

export class CurrentAccountDTO extends AccountDTO {
    @IsNumber() @Min(0) @IsOptional()
    readonly allowedOverdraft?: number
}

export class SavingsAccountDTO extends AccountDTO {
    @IsNumber() @Min(0) @IsDefined()
    readonly interestRate: number

    @IsNumber() @Min(1) @Max(31) @IsOptional()
    readonly interestApplicationDay?: number
}

export class InternationalAccountDTO extends AccountDTO {
    @IsString() @Length(5, 39) @Matches(/^[A-Z]{2}[0-9]{2}.*$/) @IsOptional()
    readonly iban?: string

    @IsString() @Length(3,3) @IsOptional()
    readonly alternateCurrency?: string
}
```

El código quedó más compacto, pero a mí personalmente, no es lo que más me interesa. Me resulta más relevante que el código es más expresivo: ahora queda claro que hay datos comunes a todos los tipos de cuenta, y que el servicio maneja tres tipos de cuenta. 

Respecto de las consecuencias, notemos dos cosas
- no hubo que cambiar _nada_ en el controller.
- la composición de definiciones usando herencia respeta los decorators. P.ej. la clase `SavingsAccountDTO` incluye la property `cbu`, con los decorators `@IsString`, `@Length` e `@IsNotEmpty`, _igual_ que si la property estuviera definida en la misma clase en lugar de incorporada por la herencia.


## A veces hay que mirar varios archivos
Tomemos otro caso: un servicio que incluye alta de oficinas y de sucursales, en dos módulos distintos. Estos son los datos para dar de alta una oficina.

``` typescript
export class AddOfficeDTO {
    @IsNumber() @Min(10) @Max(9999999)
    readonly area: number

    @IsBoolean() @IsDefined()
    readonly isSharedBuilding: boolean

    @IsNumber() @Min(0) @Max(9999) @IsOptional()
    readonly bathroomCount?: number

    @IsBoolean() @IsDefined()
    readonly allowsGeneralPublic: boolean

    @IsString() @IsNotEmpty()
    readonly department: string

    @IsInt() @Min(1) @Max(99999) @IsDefined()
    readonly internetSpeed: number
}
```

y estos los de una sucursal

``` typescript
export class AddBranchDTO {
    @IsNumber() @Min(10) @Max(9999999)
    readonly area: number

    @IsBoolean() @IsDefined()
    readonly isSharedBuilding: boolean

    @IsInt() @Min(0) @Max(9999) @IsOptional()
    readonly bathroomCount?: number

    @ApiModelProperty() @IsBoolean() @IsDefined()
    readonly allowsGeneralPublic: boolean

    @IsInt() @Min(1) @Max(99999) @IsDefined()
    readonly branchNumber: number

    @IsBoolean() @IsOptional()
    readonly opensOnSaturday?: boolean
}
```

Vemos que hay datos comunes a las oficinas y sucursales (y cualquier otro tipo de "lugar" que se quiera definir): superficie, si el edificio es compartido o no, la cantidad de baños, y si se admite el acceso al público en general.

En este caso, las clases están en archivos distintos, en carpetas distintas, porque aplican a distintos módulos. Este es un ejemplo en el que hay que tener una mirada un poco más abarcativa para encontrar las oportunidades de refactor.

Notamos también que en los decorators de `bathroomCount` hay una diferencia: en oficinas está como number, en sucursales como int. Dado que un lugar no puede tener tres baños y medio, lo correcto es int. Al refactorizar, también vamos puliendo pequeñas diferencias y errores; los refactors también son oportunidades para revisar el código.

En mi opinión, la resolución más prolija pasa por definir una carpeta _separada_ que contenga la superclase con las definiciones en común, para que no quede ni en un módulo ni en el otro. De esta forma, estamos señalando que hay un concepto común que aplica a distintos módulos.


## Los límites de la factorización
Lamentablemente, la implementación de decorators en Typescript parece ser un poco débil. Esto limita las "magias" que se pueden hacer para organizar definiciones, cuando incluyen decorators.

Por ejemplo, no se puede crear una definición a partir de otra, cambiando o agregando decorators. Por eso, si tenemos dos definiciones casi iguales, donde la única diferencia es que una tiene `@IsOptional` y la otra no, ahí tenemos que repetir.

Las limitaciones de los decoratos también nos impiden usar [mixines](https://www.typescriptlang.org/docs/handbook/mixins.html), que son un mecanismo poderoso para organizar definiciones.



## Vale definir tipos auxiliares
Agreguemos esta nueva variante de cuenta.

``` typescript
export class ExternalAccountDTO extends AccountDTO {
    @IsInt() @Min(1) @Max(999) 
    readonly dailyOperationsLimit: number

    @IsString() @Length(1,255) @IsOptional()
    readonly description?: string

    @IsOptional()
    readonly owner: {
        name: string,
        lastName: string,
        cuil: string,
        cuit: string,
        birthDate: string
    }
}
```

Si la definición de `ExternalAccountDTO` se hace muy larga, o el tipo de `owner` pudiera ser útil en otro lado, se puede separar esta definición aparte.

``` typescript
interface OwnerData {
    name: string,
    lastName: string,
    cuil: string,
    cuit: string,
    birthDate: string
}

export class ExternalAccountDTO extends AccountDTO {
    @IsInt() @Min(1) @Max(999) 
    readonly dailyOperationsLimit: number

    @IsString() @Length(1,255) @IsOptional()
    readonly description?: string

    @IsOptional()
    readonly owner: OwnerData
}
```

Las definiciones nos quedan un poco más legibles.


## Para ejercitarse
Varios casos del servicio de ejemplo sobre personas.

En `api/persons/dto/index.ts`, hay cuatro tipos de direcciones que podrían ser uno solo. Para no tocar nada en los usos, dejar las cuatro clases, se acepta extender sin agregar nada.

En `api/customer/dto/index.ts`, pasa algo similar con dos tipos de direcciones.

Para ver otro efecto positivo de los refactors: si buscan todas las definiciones de personas (buscando p.ej. `readonly lastName: string`), van a descubrir que una no se usa, y bien podría limpiarse. Un ejemplo de "código okupa".

Otro para mirar: definiciones de `Device` en `api/customer/dto/index.ts` y en `api/persons/dto/index.ts`.

En `api/sign-up-requests/dto/index.ts`, hay una estructura enorme que se llama `UpdatePersonRequestDto`. Ahí adentro hay un atributo `facephiCustomerDocumentData`. Si miran bien, adentro de ese tipo hay algo que se repite exactamente igual. Extraer los tipos de los atributos compuestos de forma tal de alivianar un poco la definición de `UpdatePersonRequestDto`, y de evitar las repeticiones (al menos la más evidente) dentro del tipo de `facephiCustomerDocumentData`.

En `api/persons/dto/index.ts` y `api/sign-up-requests/dto/index.ts`, hay dos versiones de `Pep`, y lo mismo de `ObligatedSubject`. Acá se podría armar una carpeta aparte con definiciones comunes a varios módulos.



