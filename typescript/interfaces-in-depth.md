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

Recordemos lo que dijimos antes, respecto de considerar al tipo como un filtro. Si los vemos así, los tipos `Address` y `{ street: string, streetNumber: number}` son **idénticos**: dejan pasar _exactamente_ a los mismos valores.

Incluso si definimos
``` typescript
interface StillAnotherAddress {
    street: string,
    streetNumber: number
}

const stillAnotherLongAddress = (addr: StillAnotherAddress) => {
    return addr.street.length > 30
}
```
el nuevo tipo, aunque tiene otro _nombre_, es equivalente a los anteriores.

Por eso se dice que TS adopta _tipado estructural_: lo que define un tipo está dado por la _estructura_, no por el nombre. Dos definiciones que especifican la misma estructura, son dos nombres para el mismo tipo.  
A veces se usa _duck typing_ como sinónimo para "tipado estructural". Por qué: porque en este esquema de tipado, si un animal camina como pato y hace "cuack", se lo considera pato, por más que no se lo haya especificado como pato. Cuack.  
Para dar un ejemplo del "otro barrio": Java adopta _tipado nominal_, el tipo está dado por su nombre.

[Este artículo](https://medium.com/@KevinBGreene/surviving-the-typescript-ecosystem-interfaces-and-structural-typing-7fcecd54aef5) cuenta la historia. De [este](https://medium.com/redox-techblog/structural-typing-in-typescript-4b89f21d6004) me gusta la frase "the structure _is_ the type".

Tal vez la consecuencia más importante es que se puede hacer esto
``` typescript
const isLong = longAddress({street: "Yavi", streetNumber: 338})
```
sin necesidad de especificar un tipo para el argumento. Esto hace la vida más fácil, en particular con código que viene de JS.


### OJO con any

pasa

### Algunas variantes: herencia de interfaces, atributos opcionales, readonly, interfaces abiertas

Ver que los literales son un caso especial, y que hay una compiler option especial para eso.

### Tipos función, varianzas