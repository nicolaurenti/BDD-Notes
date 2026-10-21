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

## Ejemplos de los tres alcances

Sobre una tabla ilustrativa ARTICULO:

```sql
-- Una fila como máximo si id_articulo es la PK.
DELETE FROM articulo WHERE id_articulo = 10;

-- Todas las filas del autor indicado.
DELETE FROM articulo WHERE autor = 'Ana';

-- Todas las filas, conservando la tabla.
DELETE FROM articulo;
```

Son alternativas para estudiar el alcance de la sentencia, no una secuencia ejecutada. En el segundo caso puede haber cero, una o muchas coincidencias. DELETE no significa «borrar una tabla» ni «borrar una base».

Antes de determinar el resultado, revisar si otras tablas referencian las filas seleccionadas. Si CONTIENE referencia un artículo y la relación impide su baja, no puede eliminarse ese artículo dejando asociaciones sin padre. El resultado depende de las restricciones declaradas; no se deduce únicamente del WHERE.

En papel, registrar tanto las filas elegidas como si el motor admite eliminarlas. Elegir una fila mediante una clave correcta no garantiza que su borrado sea válido.

## Fuente de estudio

Enunciado del TP1 en la carpeta `../Practicos` y filminas de cátedra `BDI_01 (Intro - creacion esquemas).pdf` (carpeta de cursada 2023).
