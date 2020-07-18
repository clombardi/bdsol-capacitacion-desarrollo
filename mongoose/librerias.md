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

## Las librerías que vamos a estudiar
En esta capacitación, trabajaremos con las dos librerías utilizadas en los proyectos de Banco del Sol. Ambas son librerías de alto nivel, las dos pertenecen al "mundo JavaScript", y en particular se llevan bien con TypeScript. 


