# Pipes
En esta página vamos a comentar el rol de los _`Pipes`_, uno de los tipos específicos de middleware que provee NestJS. La información completa sobre este tema se puede consultar a partir [la página correspondiente en la documentación de NestJS](https://docs.nestjs.com/pipes).

El objetivo de los Pipes es hacer validaciones, y en algunos casos también transformaciones, sobre los _datos de entrada_, o sea los que llegan en el request: body, headers, parámetros de path y de query.

Si los datos no cumplen con las condiciones requeridas por un Pipe, el código del request handler no se llega a ejecutar, y se genera una respuesta con status code `400 - Bad Request`. El Pipe es el encargado de generar el mensaje de error correspondiente.  
Como ya vimos al revisar el [manejo de errores](./manejo-de-errores.md), en realidad lo que hace el Pipe es lanzar una excepción `BadRequestException`, con un determinado mensaje. Esta excepción puede ser gestionada por un `ExceptionHandler`, que puede realizar las transformaciones y acciones adicionales que se estimen convenientes.

_NestJS provee varias implementaciones operativas de Pipe_, que se pueden integrar en nuestras aplicaciones.  
También se permite la definición de _Pipes particulares_ (custom Pipes). 
Más abajo en esta página daremos un ejemplo de implementación de un Pipe para una validación/transformación particular.
Para más detalles sobre la implementación de Pipes particulares, remitimos a [la documentación](https://docs.nestjs.com/pipes). 


## ValidationPipe
Tal vez, el Pipe más valioso que provee NestJS es el `ValidationPipe`, que se describe en una [página separada](https://docs.nestjs.com/techniques/validation) en la documentación.  
Este componente es utilizado en varios microservicios del Banco del Sol.

Este Pipe funciona en conjunto con el package [class-validator](https://github.com/typestack/class-validator). En particular, resulta útil para validar request bodies.
Para utilizarlo, debe decorarse la clase que modela al body correspondiente, utilizando decorators provistos por `class-validator`.  
El `ValidatorPipe` utiliza las capacidades de validación de este package, y lanza la excepción correspondiente si los datos recibidos en un request no cumplen con las condiciones indicadas en los decorators.  
Por lo tanto, sólo hace falta: decorar la clase que modela el body, y activar el `ValidatorPipe` en el request handler.

Mostramos un ejemplo, que se requiere a un módulo en el que se registran gastos. Este módulo incluye un endpoint para agregar un gasto, al que decoramos para aplicarle el `ValidationPipe`.
``` typescript
@Post()
@UsePipes(new ValidationPipe())
addExpense(@Body() expenseData: AddExpenseRequestDTO): AddExpenseResponseDTO {
    // implementación
}
```

Las validaciones se realizan decorando la clase `AddExpenseRequestDTO`.
``` typescript
import { IsNotEmpty, IsNumber, IsString, IsOptional, Min, MinLength, IsDefined } from 'class-validator';

export class AddExpenseRequestDTO {
    @IsDefined({ message: 'Una fecha hay que poner che' }) 
    expenseDate: string
    @IsNotEmpty({ message: 'Es obligatorio indicar un responsable' })
    responsible: string
    @IsOptional() @IsString() @MinLength(5)
    description: string
    @IsNumber() @Min(0)
    amount: number
    @Allow()
    comments: string
}
```

Este es un ejemplo de request que genera varios errores de validación.
![request con tres errores](./images/validaton-pipe-three-errors.jpg)

Aquí se ve que el `message` que genera `ValidationPipe` es extenso, y que se trata de una _lista_, en la que aparecen todos los errores detectados. Será función de un `ExceptionFilter` el extraer y manipular esta información para generar la response.
También se ve que los decorators de `class-validator` pueden ser configurados con un mensaje específico, proveyendo mensajes por defecto si no se configuran.

### Campos requeridos, opcionales y no definidos
Aunque entendemos que todos los decorators son autoexplicativos, preferimos hacer algunas puntualizaciones sobre `@IsDefined` e `@IsOptional`.

En el ejemplo anterior, el error del atributo `responsible` se generó por el decorator `@IsNotEmpty`: un atributo no incluido se considera vacío. Por otro lado, la ausencia del atributo `comments` no generó un error.  
Para pedir que un atributo sea incluido, sin hacer ninguna indicación sobre el valor, está el decorator `@IsDefined`. Es el caso del atributo `expenseDate` en el ejemplo.
![error de IsDefined](./images/validaton-pipe-is-defined-error.jpg)

El atributo `@IsOptional` deshabilita todos los chequeos si el atributo no está presente. Es el caso de `description` en el ejemplo.
![campo opcional puede estar ausente](./images/validaton-pipe-optional-field.jpg)

Obsérvese la diferencia con el campo `responsible` en el primer ejemplo de request: al no tener valor, se considera vacío y se aplica la validación. Por otro lado, el campo `comments`, que no tiene ninguna validación (el `@Allow` sólo "registra" el atributo para `class-validator`), puede o no estar.

Para este ejemplo, el comportamiento del servicio es responder con el request.
``` typescript
@Post()
@UsePipes(new ValidationPipe())
addExpense(@Body() expenseData: AddExpenseRequestDTO): AddExpenseRequestDTO {
    return {...expenseData, amount: expenseData.amount + 1 }
}
```
Esta definición nos va a permitir analizar el comportamiento de `ValidationPipe` si el request incluye atributos que no están especificados en el DTO.  
El comportamiento por defecto es no rechazar el request y hacer llegar los datos "sobrantes" al request handler.
![incluye dato no especificado](./images/validaton-pipe-not-whitelisted.jpg)

El `ValidationPipe` se puede configurar, pasando un parámetro en el constructor que incluye los atributos de configuración.  
Uno de estos atributos es `whitelist`, su efecto es que los datos no definidos en el DTO no lleguen al request handler. O sea, esta definición
``` typescript
@Post()
@UsePipes(new ValidationPipe({ whitelist: true }))
addExpense(@Body() expenseData: AddExpenseRequestDTO): AddExpenseRequestDTO {
    return {...expenseData, amount: expenseData.amount + 1 }
}
```
tiene el siguiente efecto
![whitelist](./images/validaton-pipe-whitelist.jpg)

Si **además** de `whitelist`, se agrega `forbidNonWhitelisted`, entonces se rechazarán los request con datos no previstos. Por ejemplo, esta definición 
``` typescript
@Post()
@UsePipes(new ValidationPipe({ whitelist: true, forbidNonWhitelisted: true }))
addExpense(@Body() expenseData: AddExpenseRequestDTO): AddExpenseRequestDTO {
    return {...expenseData, amount: expenseData.amount + 1 }
}
```
genera este resultado
![forbid not whitelisted](./images/validaton-pipe-forbid-not-whitelist.jpg)

También mencionamos el atributo `skipMissingProperties`, que transforma todos los atributos en opcionales, excepto los que tengan marcado explícitamente `@IsDefined`. Si definimos
``` typescript
@Post()
@UsePipes(new ValidationPipe({ skipMissingProperties: true }))
addExpense(@Body() expenseData: AddExpenseRequestDTO): AddExpenseRequestDTO {
    return {...expenseData, amount: expenseData.amount + 1 }
}
```
podemos no incluir p.ej. el atributo `responsible`, que generó un error en el primer request.
![skip missing properties](./images/validaton-pipe-skip-missing-properties.jpg)


### Validaciones particulares
Una validación interesante es `@Matches`, que recibe una [expresión regular](https://docs.python.org/3/library/re.html) como parámetro. Por ejemplo, para validar que la fecha tenga formato `YYYY-MM-DD` (sin validar que en rigor sea una fecha válida), se puede especificar de esta forma
``` typescript
    @IsDefined({ message: 'Una fecha hay que poner che' }) 
    @Matches(/[0-9]{4}-[0-9]{2}-[0-9]{2}/, {message: 'No cumple el formato de fecha'})
    expenseDate: string
```
y se comporta como uno espera
![regex](./images/validaton-pipe-regex.jpg)

Finalmente, mencionamos que se pueden definir _custom validators_, ver los detalles en la [documentación de class-validator](https://github.com/typestack/class-validator).


### Comentario - integración de packages
Una característica que creo muy positiva de NestJS es que confía en otros packages para algunas tareas que integra en el framework.  
Desde el principio, no pretende implementar un Web server, para eso confía en Express; se concentra en dar un modelo que facilita y organiza el desarrollo.  
La integración con `class-validator` es otro ejemplo: no repite funcionalidad que está desarrollada en otro package, sino que lo integra a los conceptos que define para brindar una solución de validación sencilla y potente.


## Pipes de tipos particulares



## Desafíos

### Valores en un `enum`
Definir un `enum` para los valores posibles de `responsible`. Usar la validación `@IsEnum` para validar que el valor que llega en el request es correcto.

### Descuento menor al importe
Agregar un atributo `discount`, que si está, su valor tiene que ser menor al de `amount`. Definir un custom validator para esto.