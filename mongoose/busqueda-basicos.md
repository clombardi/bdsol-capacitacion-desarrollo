# Búsqueda de documentos
En la página anterior, vimos que el resultado de la operación `<modelo>.find()`, _sin parámetros_, es un array que incluye _todos_ los documentos de la colección correspondiente.

A la misma operación se le pueden pasar _criterios de búsqueda_, para restringir los documentos que se obtienen.  
Los mismos criterios se pueden aplicar a la operación `findOne()`, y también a otras operaciones similares como p.ej. `findOneAndDelete()`. Ver el listado de [operaciones sobre un modelo](https://mongoosejs.com/docs/api/model.html).

