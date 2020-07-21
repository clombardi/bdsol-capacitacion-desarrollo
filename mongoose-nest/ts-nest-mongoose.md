# TS + Mongoose + Nest
En esta página y las que siguen, vamos a integrar las tres tecnologías principales con las que trabajamos hasta ahora: TypeScript, NestJS y Mongo/Mongoose.

En particular, vamos a hacer práctica sobre
- la definición de tipos en TypeScript.
- los conceptos básicos de Nest: módulo, controller, provider, request handler.
- los conceptos básicos de Mongoose: conexión, esquema, modelo, operación.

En concreto, vamos a definir un módulo Nest que responde requests a partir de información que está en una BD Mongo, a la que se accede mediante Mongoose.

En particular, vamos a usar el soporte que da Nest para usar Mongoose, que se va a encargar de la conexión y de la creación de modelos.  

> **Atención**  
> Vamos a basarnos en el soporte para Mongoose de la _versión 6_ de Nest, que lamentablemente no tiene link directo, hay que ir desde [el inicio de la doc de la versión 6](https://docs.nestjs.com/v6/), y de ahí a Techniques -> Mongo.

Vamos a intentar maximizar el uso de tipos, o sea, a no dejar `any` en ningún lado, y hacer la menor cantidad de casteos posible.  
Tenemos dos motivaciones para esto. 
Una es observar cómo pueden aplicarse tipos al trabajar con Mongoose desde una apliación codificada en TypeScript.
La otra es más general: analizar las diferencias que aparecen entre los tipos de las entidades que manejamos, entre la API, el servicio, y la base de datos.

Esto no quiere decir que se recomiende tipar todo, todo el tiempo; la decisión de maximizar el tipado se tomó a efectos demostrativos.

Debemos recordar que para usar una versión tipada de Mongoose debemos incorporar el package `@types/mongoose`, o sea.
```
> npm install @types/mongoose --save-dev
```

## El ejemplo - solicitudes de cuenta
Vamos a usar el mismo dominio de solicitudes de cuenta con el que practicamos [el uso de Mongoose sobre JavaScript](../mongoose/mongoose-cuatro-conceptos.md). Podemos incluso consultar las mismas solicitudes que agregamos usando JavaScript.

Para arrancar, hay que definir un nuevo módulo en una aplicación Nest, con su controller y su service.  
Para construir sobre algo ya armado, armemos un endpoint `@Get` que devuelve una lista fija de `AccountRequest`s que se define en el servicio (para ya tener armado el camino controller -> service).  
Pueden dejar la fecha para más adelante ... o no, como prefiera cada une.