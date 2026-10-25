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

## Distinguir los dos criterios de la consigna

La consigna pide justificar por separado con los criterios SQL-1999 de las filminas y las condiciones de PostgreSQL. Proyectar la PK y usar un join no reciben el mismo tratamiento en ambos análisis.

### Comparar actualizabilidad SQL-1999 y PostgreSQL

Reglas de la cátedra, tabla de construcciones y método para responder ambos apartados.

#### Criterio de cátedra y comportamiento PostgreSQL

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

Las filminas enumeran también LIMIT/OFFSET y otras construcciones entre las limitaciones de actualización automática en PostgreSQL.

#### Justificar con las reglas que pide la consigna

Para el análisis **según SQL-1999 presentado por la cátedra**, comprobar en una vista simple: conservación de todas las columnas de la PK, ausencia de DISTINCT, agregados e información derivada y ausencia de subconsultas en la lista SELECT. Una subconsulta en WHERE no es lo mismo que una subconsulta que produce una columna. En vistas de ensamble se analiza, además, preservación de clave.

Para el apartado **PostgreSQL**, responder por separado: una única tabla o vista actualizable en FROM, sin las construcciones que impiden actualización automática en el nivel principal. No trasladar al estándar la excepción del motor sobre proyectar la PK, ni afirmar que todos los motores admiten automáticamente una vista de join.

## Validez de la escritura concreta

La actualizabilidad de la vista y la validez de una escritura son preguntas distintas. Una operación puede fallar por obligatoriedad, unicidad o referencia aunque la vista admita ese tipo de operación.

### Ejemplo de INSERT rechazado en una vista actualizable

Caso de EMPLEADO_DIST_20 con una columna base obligatoria que la vista no proyecta.

#### Una vista actualizable puede rechazar un INSERT

Supongamos EMPLEADO_DIST_20 con id, nombre, apellido, sueldo y nacimiento, pero sin la columna id_distribuidor, y que esta es NOT NULL sin DEFAULT en EMPLEADO. La vista puede permitir modificar el sueldo de una fila existente, pero un INSERT no aporta el distribuidor y se rechaza por esa obligatoriedad. El filtro `id_distribuidor = 20` no asigna automáticamente 20 a una columna omitida.

También puede rechazarse un INSERT porque la PK ya existe o una FK referencia un padre inexistente. La respuesta completa distingue **estructura de la vista** y **validez de la operación concreta**. «Es actualizable» contesta solo la primera parte.

## Fuente de estudio

Enunciado del TP5 en la carpeta `../../Practicos` y filminas de cátedra `BDI_05 (Vistas).pdf` (carpeta de cursada 2023).
