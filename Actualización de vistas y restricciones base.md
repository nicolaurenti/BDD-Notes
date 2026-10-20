---
tags:
  - tp5
---

# Actualización de vistas y restricciones base

## Leer una vista y actualizarla son problemas distintos

Para una escritura sobre una vista hay que decidir:

1. Si existe traducción automática hacia las tablas base.
2. Qué columnas se pueden escribir y cuáles son calculadas.
3. Si la operación respeta PK, FK, CHECK, NOT NULL y valores por defecto.
4. Si debe satisfacer un `WITH CHECK OPTION`.

Que una vista sea actualizable no garantiza que cualquier INSERT funcione. Por ejemplo, una columna obligatoria omitida en la vista puede impedir insertar si no tiene un valor por defecto.

## Criterio de cátedra y comportamiento PostgreSQL

Las filminas presentan como criterio para vistas simples: conservar la clave, evitar DISTINCT, agregaciones, información derivada y subconsultas en la lista SELECT. Para ensambles, analizan preservación de clave mediante relaciones FK→PK.

No trasladar esas reglas sin más al motor: en PostgreSQL una vista simple puede ser automáticamente actualizable aunque no proyecte toda la PK; la operación concreta todavía debe poder satisfacer las restricciones de la base.

| Construcción | Guía para PostgreSQL |
|---|---|
| Una tabla o vista actualizable en FROM; columnas directas | Habitualmente actualizable |
| Subconsulta en WHERE con columnas directas de una sola tabla exterior | Puede ser actualizable |
| JOIN de varias tablas en FROM | No se actualiza automáticamente |
| DISTINCT, GROUP BY, HAVING o agregados en el nivel principal | No se actualiza automáticamente |
| UNION, INTERSECT o EXCEPT en el nivel principal | No se actualiza automáticamente |
| Columna calculada | Esa columna no es automáticamente escribible; puede haber otras que sí lo sean |

También hay limitaciones con LIMIT/OFFSET, ventanas y otras construcciones. Ver [condiciones de vistas actualizables en PostgreSQL](https://www.postgresql.org/docs/current/sql-createview.html#SQL-CREATEVIEW-UPDATABLE-VIEWS).
