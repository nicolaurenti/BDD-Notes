---
tags:
  - tp3
---

# Fundamentos y clasificación de restricciones

## Alcance y clasificación de restricciones

Una restricción expresa qué estados de la base son válidos. No alcanza con controlar la aplicación: el motor debe rechazar las operaciones que dejan datos inconsistentes.

| Alcance usado por la cátedra | Qué compara | Ejemplo | Recurso |
|---|---|---|---|
| Atributo o dominio | Un valor | Nacionalidad dentro de una lista | `CHECK`, `DOMAIN`, `NOT NULL` |
| Tupla | Columnas de la misma fila | Descripción y presentación no ambas nulas | `CHECK` de fila |
| Tabla | Varias filas de una tabla | Hasta 15 palabras por artículo en `contiene` | CHECK de tabla en SQL-1999; no implementable con subconsulta en PostgreSQL |
| General | Datos de varias tablas | Horas del voluntario dentro de los límites de su tarea | `ASSERTION` en SQL-1999; no disponible en PostgreSQL |

La clasificación depende del modelo: identificar primero dónde están los atributos y qué filas hay que consultar. Un `CHECK` escrito al final del `CREATE TABLE` no se vuelve, por esa ubicación, un control de múltiples filas.

### Atributo y tupla

Una regla de atributo controla un valor; una de tupla relaciona columnas de la misma fila. CHECK y DOMAIN permiten expresar esos controles sin consultar otras filas.

[[TP3 - Ejemplos de restricciones de atributo y tupla|Ejemplos de restricciones de atributo y tupla]]

### Tabla

Una regla de tabla compara distintas filas de la misma relación. UNIQUE expresa unicidad; un máximo por grupo requiere un control agregado en el modelo declarativo del estándar.

[[TP3 - Ejemplos de restricciones de tabla|Ejemplos de restricciones de tabla]]

### General

Una regla general consulta varias tablas. El ejemplo compara las horas del voluntario con los límites de su tarea mediante una ASSERTION de SQL-1999.

[[TP3 - Ejemplo de restricción general entre Voluntario y Tarea|Ejemplo de restricción general entre Voluntario y Tarea]]

### Cómo elegir el alcance

1. ¿La regla se decide mirando un solo atributo? **Atributo o dominio.**
2. ¿Necesito varios atributos, pero solo de la misma fila? **Tupla.**
3. ¿Necesito otras filas de la misma tabla? **Tabla.**
4. ¿Necesito consultar varias tablas? **General.**

Primero determinar el alcance; después elegir el recurso que lo expresa. El nombre de la sentencia no determina por sí solo la clasificación: una ASSERTION puede expresar una regla que consulta una sola tabla, y una restricción escrita al final de CREATE TABLE puede seguir siendo de tupla.

Los fragmentos son ejemplos de estudio: no se ejecutaron ni se crearon restricciones en las bases existentes.

## PK, UNIQUE, NOT NULL y FK

PK, UNIQUE, NOT NULL y FK son recursos declarativos específicos. Se repasan sus garantías antes de elegir controles más generales.

[[TP3 - Repaso de PK, UNIQUE, NOT NULL y FK|Repaso de PK, UNIQUE, NOT NULL y FK]]

## Fuente de estudio

Enunciado del TP3 en la carpeta `../Practicos` y filminas de cátedra `BDI_03.P1 (Restricciones-1).pdf; BDI_03.P1 (Restricciones-2).pdf` (carpeta de cursada 2023).
