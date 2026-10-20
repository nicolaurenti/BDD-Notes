---
tags:
  - tp3
---

# Alcance y clasificación de restricciones


Una restricción expresa qué estados de la base son válidos. No alcanza con controlar la aplicación: el motor debe rechazar las operaciones que dejan datos inconsistentes.

| Alcance usado por la cátedra | Qué compara | Ejemplo | Recurso |
|---|---|---|---|
| Atributo o dominio | Un valor | Nacionalidad dentro de una lista | `CHECK`, `DOMAIN`, `NOT NULL` |
| Tupla | Columnas de la misma fila | Descripción y presentación no ambas nulas | `CHECK` de fila |
| Tabla | Varias filas de una tabla | Hasta 15 palabras por artículo en `contiene` | Chequeo de tabla en el modelo estándar; trigger si requiere agregación en PostgreSQL |
| General | Datos de varias tablas | Horas del voluntario dentro de los límites de su tarea | `ASSERTION` en el estándar; alternativa procedural en PostgreSQL |

La clasificación depende del modelo: identificar primero dónde están los atributos y qué filas hay que consultar. Un `CHECK` escrito al final del `CREATE TABLE` no se vuelve, por esa ubicación, un control de múltiples filas.
