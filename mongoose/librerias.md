---
layout: default
---

# Librerías de acceso a Bases de Datos
El acceso a bases de datos desde una aplicación, se hace (al menos casi-casi-siempre) mediante _librerías_.  

Algunas de estas librerías son básicamente conectores que permiten ejecutar las operaciones de BD desde un programa, digamos que son librerías de _bajo nivel_. Un ejemlo es la venerable librería [JDBC](https://www.javatpoint.com/java-jdbc) de Java.  

Hay otras librerías de más _alto nivel_, que incluyen funcionalidades que simplifican la interacción con bases de datos, en distintos aspectos que pueden incluir
- el modelado de datos, incluyendo validaciones y transformaciones.
- el manejo de transacciones.
- el manejo de sesiones.
- la definición de caches.
- herramientas para tests.

Por otra parte, cada librería está orientada a un _lenguaje de programación_, o a varios lenguajes específicos.


## ORM y ODM
Entre las librerías de alto nivel para acceso a bases de datos _relacionales_, se destacan las ORM (Object-Relational Mapper). Este concepto está establecido desde hace más de 15 años; existen muchas librerías ORM para distintos lenguajes. Tal vez una de las más conocidas, al menos desde el punto de vista histórico, es [Hibernate](https://hibernate.org/orm/).  
Las ORM se encargan específicamente de las transformaciones necesarias para "traducir" una base de datos relacional a un modelo de objetos, y viceversa; lo que resulta complejo dado que los modelos no son similares.
Estas librerías suelen ser compatibles con distintas bases de datos relacionales, aprovechando el alto grado de estandarización de este modelo de bases de datos.

Por su parte, las librerías ODM (Object Data Modelling) se encargan de la interface con bases de datos _de documentos_. Su tarea es menos compleja que la de los ORM, porque el modelo de una base de datos de documentos es más similar a lo que manejan muchos programas en JS / TS.  
El concepto de ODM es más reciente que el de ORM; aparece junto con la irrupción de las bases de documentos en la industria.  
Al contrario de los ORM, los ODM suelen ser específicos para una base de datos particular. 



## Las librerías que vamos a estudiar
En esta capacitación, trabajaremos con las dos librerías utilizadas en los proyectos de Banco del Sol. Ambas son librerías de alto nivel, las dos pertenecen al "mundo JavaScript", y en particular se llevan bien con TypeScript. 

Para bases de datos de documentos, la librería es [Mongoose](https://mongoosejs.com/), que es tal vez es la librería ODM más popular; ver en su [página de npm](https://www.npmjs.com/package/mongoose) y en [esta comparación entre ODM](https://openbase.io/packages/top-javascript-mongodb-odm-libraries).
Mongoose es específica para MongoDB.

Para bases de datos relacionales, vamos a estudiar [TypeORM](https://typeorm.io/), una librería ORM para JS / TS.


