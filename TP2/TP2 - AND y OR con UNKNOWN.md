---
tags:
  - tp2
---

# TP2 - AND y OR con UNKNOWN

Tabla de combinaciones y un caso de filtro que incluye valores ausentes.

## Cómo se combina UNKNOWN con AND y OR

El resultado se puede decidir cuando la otra condición alcanza para determinarlo. En `FALSE AND UNKNOWN`, ya sabemos que no se cumplen ambas: da FALSE. En `TRUE OR UNKNOWN`, ya sabemos que al menos una se cumple: da TRUE.

| A | A AND UNKNOWN | A OR UNKNOWN |
|---|---|---|
| TRUE | UNKNOWN | TRUE |
| FALSE | FALSE | UNKNOWN |
| UNKNOWN | UNKNOWN | UNKNOWN |

Por ejemplo, para horas NULL, `horas_aportadas > 1000 OR horas_aportadas IS NULL` da `UNKNOWN OR TRUE`, o sea TRUE: se incluye. En cambio, `NOT(horas_aportadas > 1000)` sigue dando UNKNOWN y la fila queda fuera.

Antes de agregar COALESCE, decidir qué significa el dato ausente. Un promedio de horas conocidas y un promedio que considera cero las horas desconocidas responden preguntas distintas. El reemplazo sirve para expresar una decisión, no para borrar automáticamente todos los nulos de una consulta.

## Fuente de estudio

Enunciado del TP2 en la carpeta `../../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
