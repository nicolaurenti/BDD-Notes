---
tags:
  - tp2
---

# Estructura, ejecución y presentación de resultados

## Estructura general de una consulta

```POSTGRESQL
SELECT columnas
FROM tabla
WHERE condicion
GROUP BY columnas
HAVING condicion_sobre_grupos
ORDER BY columnas
LIMIT cantidad
OFFSET desplazamiento;
```

| Cláusula      | Función                                          |
| ------------- | ------------------------------------------------ |
| `SELECT`      | Define las columnas o expresiones del resultado. |
| `FROM`        | Indica las tablas de origen.                     |
| `JOIN ... ON` | Combina filas de tablas según una condición.     |
| `WHERE`       | Filtra filas antes del agrupamiento.             |
| `GROUP BY`    | Forma grupos de filas.                           |
| `HAVING`      | Filtra grupos.                                   |
| `ORDER BY`    | Ordena el resultado.                             |
| `LIMIT`       | Limita la cantidad de filas devueltas.           |
| `OFFSET`      | Omite una cantidad de filas.                     |

## Orden lógico de ejecución

Las filas de origen se filtran antes de agrupar; HAVING filtra grupos y SELECT produce la salida. El orden lógico explica qué expresiones se pueden usar en cada cláusula.

[[TP2 - Orden lógico de una consulta paso a paso|Orden lógico de una consulta paso a paso]]

## DISTINCT

DISTINCT elimina repeticiones de la fila completa seleccionada. No elige un valor distinto por columna ni corrige una unión mal planteada.

[[TP2 - DISTINCT y combinaciones de columnas|DISTINCT y combinaciones de columnas]]

## Ordenamiento, primeros N y paginación

ORDER BY define la presentación. LIMIT y OFFSET seleccionan una porción de ese resultado; una clave adicional permite resolver empates y paginar de forma estable.

[[TP2 - Ordenamiento, primeros N y paginación|Ordenamiento, primeros N y paginación]]

## Caso trabajado de agrupamiento

El caso de instituciones y horas permite comprobar cómo WHERE y HAVING cambian el resultado.

[[TP2 - Orden lógico de una consulta paso a paso|Orden lógico de una consulta paso a paso]]

## Fuente de estudio

Enunciado del TP2 en la carpeta `../../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
