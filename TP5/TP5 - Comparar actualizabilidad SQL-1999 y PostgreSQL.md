---
tags:
  - tp5
---

# TP5 - Comparar actualizabilidad SQL-1999 y PostgreSQL

Reglas de la cátedra, tabla de construcciones y método para responder ambos apartados.

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

Las filminas enumeran también LIMIT/OFFSET y otras construcciones entre las limitaciones de actualización automática en PostgreSQL.

## Justificar con las reglas que pide la consigna

Para el análisis **según SQL-1999 presentado por la cátedra**, comprobar en una vista simple: conservación de todas las columnas de la PK, ausencia de DISTINCT, agregados e información derivada y ausencia de subconsultas en la lista SELECT. Una subconsulta en WHERE no es lo mismo que una subconsulta que produce una columna. En vistas de ensamble se analiza, además, preservación de clave.

Para el apartado **PostgreSQL**, responder por separado: una única tabla o vista actualizable en FROM, sin las construcciones que impiden actualización automática en el nivel principal. No trasladar al estándar la excepción del motor sobre proyectar la PK, ni afirmar que todos los motores admiten automáticamente una vista de join.

## Fuente de estudio

Enunciado del TP5 en la carpeta `../../Practicos` y filminas de cátedra `BDI_05 (Vistas).pdf` (carpeta de cursada 2023).
