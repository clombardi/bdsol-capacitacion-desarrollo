## Algo más sobre interfaces

### Tipado estructural

Miremos estas definiciones

``` typescript
interface Address {
    street: string,
    streetNumber: number
}

const isLongAddress = (addr: Address) => {
    return addr.street.length > 30
}

const isLongAddress2 = (addr: { street: string, streetNumber: number}) => {
    return addr.street.length > 30
}
```

¿qué restricciones cada una? ... tipado estructural ... duck typing ...

### OJO con any

pasa

### Algunas variantes: herencia de interfaces, atributos opcionales, readonly, interfaces abiertas

Ver que los literales son un caso especial, y que hay una compiler option especial para eso.

### Tipos función, varianzas