# Guards

Un _Guard_ es un middleware pensado para implementar control de acceso. Si un request no cumple con lo requerido por un Guard que está activado para ese endpoint, entonces el request **no llega** al request handler, y se lanza una excepción.

Tal vez es más sencillo contarlo a partir de un ejemplo. En varios de los endpoints que brindan datos sobre un país, queremos impedir que se soliciten datos sobre países "sensibles".
Esta es una implementación de Guard que realiza este control.
``` typescript
const dangerousCountries = ["CHN", "AUT"]

@Injectable()
export class ForbidDangerousCountries implements CanActivate {
    canActivate(context: ExecutionContext): boolean {
        const request = context.switchToHttp().getRequest();
        const country = request.params.countryCode
        const isDangerousCountry = dangerousCountries.includes(country)
        return !isDangerousCountry
    }
}
``` 
Como vemos, un Guard es una clase que debe implementar la interface `CanActivate` e incluir el decorator `@Injectable`. La interface define un único método que devuelve un booleano. Si el método devuelve `false`, quiere decir que el Guard no acepta al request.  
Dentro del método, se accede al request mediante el `ExecutionContext` que llega por parámetro. Este `ExecutionContext` es similar al `ArgumentsHost` que recibe un `ExceptionFilter`, que se describe en [la página sobre manejo de errores](./manejo-de-errores.md).  
Los detalles sobre la implementación de Guards pueden consultarse en [la página de Guards en la documentación de NestJS](https://docs.nestjs.com/guards).