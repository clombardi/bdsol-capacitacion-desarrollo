---
layout: default
---

# Swagger en NestJS - relación con la especificación
Observemos esta especificación, que corresponde a un atributo dentro de una response
``` typescript
    @ApiPropertyOptional({ 
        description: 'How many positive opinions are required to approve the request', 
        type: 'number', minimum: 2, maximum: 1000, example: 5, default: 3
    })
    requiredApprovals: number
```
Los atributos `description`, `type`, etc., se corresponden _exactamente_ con atributos de la descripción de una `property` en la especificación Swagger. 
La definición del mismo atributo en un YAML Swagger tiene esta forma.
``` yaml
requiredApprovals:
    description: How many positive opinions are required to approve the request
    type: number
    maximum: 1000
    minimum: 2
    example: 5
    default: 3
```

Por lo general, la correspondencia entre atributos de los decorators NestJS y atributos de la especificación Swagger se mantiene.  

Pero hay algunas excepciones. Por ejemplo, para especificar que un atributo es de `type: array`, se incluye en el decorator Swagger el atributo `isArray: true`, y se indica como `type`, el tipo _de cada elemento del array_.  
Este es el caso de la respuesta del `GET` que trabajamos en la [página anterior](./swagger-nestjs-empezamos).
``` typescript
    @ApiResponse({ 
        status: HttpStatus.OK, description: 'Data delivered', 
        type: AccountRequestDTO, isArray: true 
    })
```
esto en un YAML Swagger se ve así.
``` yaml
responses:
    '200':
        description: Data delivered
        content:
            application/json:
                schema:
                    type: array
                    items:
                        $ref: '#/components/schemas/AccountRequestDTO'
```

En la descripción mediante un decorator, además de ahorrarnos los atributos intermedios `content`, `application/json` y `schema` (que son generados por NestJS automáticamente), el `type` es `array`, y lo que en NestJS se indica como `type`, para la especificación Swagger son los `items`.


## La correspondencia nos sirve para deducir cómo manejarnos en casos menos usuales
La correspondencia entre atributos de los decorators de NestJS y de la especificación Swagger, puede servir como punto de partida para descubrir la forma de especificar alguna característica menos habitual, para que se incorpore al Swagger.  
Se puede proceder de esta forma: se averigua cómo se hace en Swagger para documentar el caso en cuestión, se prueba usando p.ej. el Swagger Editor, se verifica si los mismos atributos encontrados en la especificación Swagger, son aceptados por los decorators NestJS.

Esta aclaración puede ser útil porque la documentación del soporte para Swagger de NestJS es bastante pobre, y googleando tampoco se encuentra mucho. Sobre Swagger hay mucho más material disponible.


## Endpoint que devuelve la documentación
Uno de los detalles que está mencionado al pasar en la doc de NestJS, es que además de la UI, se agrega un endpoint que devuelve el documento Swagger en formato JSON.  
Este endpoint es (obviamente) un `GET`, el path es `/<path UI>-json`, en el ejemplo de la página anterior queda `/apidoc-json`.

Con la aplicación levantada, se puede acceder p.ej. mediante Postman.
