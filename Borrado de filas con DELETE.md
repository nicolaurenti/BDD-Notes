---
tags:
  - tp1
---

# Borrado de filas con DELETE


Se utiliza `DELETE FROM tabla WHERE condición`.

- **Una fila:** filtrar por su clave.
- **Un subconjunto:** utilizar una condición que seleccione las filas.
- **Todas las filas:** omitir `WHERE`.

Borrar filas conserva la estructura de la tabla.

Las FK pueden impedir el borrado de una fila que está siendo referenciada, dependiendo de las acciones referenciales definidas.
