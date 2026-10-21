---
tags:
  - tp2
---

# TP2 - Ejercicio de Equipo con todos los puntos nulos

Caso trabajado del ejercicio 2.1 del TP2, con el resultado y la causa de cada consulta.

## Ejercicio 2.1: cuatro equipos con puntos y descripción nulos

Datos de partida: cuatro equipos con identificadores 1, 2, 3 y 4; tanto `puntos` como `descripcion` son `NULL`.

| Consulta o expresión | Resultado | Motivo |
|---|---|---|
| `AVG(puntos), COUNT(puntos), COUNT(*)` | `NULL, 0, 4` | No hay puntos no nulos, pero hay cuatro filas. |
| `SELECT DISTINCT puntos FROM equipo` | Una fila con `NULL` | `DISTINCT` reúne los nulos en un único resultado. |
| `'Su descripción es ' || descripcion` | `NULL` en cada fila | La concatenación con `||` propaga el nulo. |
| `WHERE puntos NOT IN (SELECT DISTINCT puntos FROM equipo)` | Ninguna fila | `NULL NOT IN (NULL)` es `UNKNOWN`. |
| Autojoin con `ON e1.puntos = e2.puntos` | Ninguna fila | Todas las comparaciones son `NULL = NULL`, que da `UNKNOWN`. |

> [!important]
> La consulta 2 de la imagen también contiene el filtro `NOT IN`. Por eso la consulta completa no devuelve filas. Solo si se quita ese filtro aparecen los cuatro identificadores con la descripción concatenada en `NULL`.

## Fuente de estudio

Enunciado del TP2 en la carpeta `../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
