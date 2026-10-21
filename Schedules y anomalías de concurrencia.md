---
tags:
  - tp8
---

# Schedules y anomalías de concurrencia


`r1(A)` significa que T1 lee A; `w1(A)` que lo escribe; `c1` confirma T1 y `a1` la aborta. Un schedule intercala operaciones respetando el orden interno de cada transacción.

| Problema | Qué ocurre |
|---|---|
| Actualización perdida | Una escritura calculada con un valor viejo pisa un cambio ajeno |
| Lectura sucia | Se lee una escritura de otra transacción todavía no confirmada |
| Lectura no repetible | Se relee una fila y otra transacción confirmó un cambio entre ambas lecturas |
| Fantasma | Al repetir un predicado cambia el conjunto de filas por escrituras ajenas |
| Resumen inconsistente | Un cálculo mezcla partes de estados de momentos distintos |

## Casos trabajados del banco

Para demostrar una anomalía hace falta escribir el orden de lecturas, cálculos, escrituras y confirmaciones que la produce.

[[TP8 - Ejemplos del banco de actualización perdida y lectura sucia|Ejemplos del banco de actualización perdida y lectura sucia]]

## Armar el intercalado que demuestra el problema

Para demostrar actualización perdida, mostrar que ambas transacciones leen el mismo saldo viejo, calculan resultados distintos y una escritura pisa a la otra. No basta con decir «se ejecutan juntas». La operación de lectura, el cálculo local y la escritura deben ubicarse en el schedule.

Para demostrar lectura sucia en el banco, T1 escribe el saldo debitado 5000, T2 lo lee antes de que T1 confirme y después T1 aborta. T2 usó un valor que no formará parte del estado confirmado. Si T2 ya confirmó un resultado dependiente, la dificultad de recuperación es mayor.

Una lectura no repetible requiere dos lecturas de la misma fila por la misma transacción, con un cambio confirmado entre ellas. Un fantasma requiere repetir una consulta por condición y obtener un conjunto de filas diferente: por ejemplo, otro movimiento incorpora una cuenta que ahora entra en el total.

## Serial no es lo mismo que correcto en el orden esperado

Un schedule serial T1 seguido de T3 y otro T3 seguido de T1 pueden ser ambos admisibles si cada transacción preserva las reglas. En el cliente 10, restar 5000 y sumar 15000, desde 10000, termina en 20000 en ambos órdenes. Un schedule que termine en 5000 o 25000 por pisar una escritura muestra una pérdida, porque no refleja ambas operaciones.

Cuando se propone un intercalado, conservar siempre el orden interno de cada transacción: no colocar su escritura antes de su lectura si su programa las realiza al revés. Ese es el límite que define cuáles schedules son posibles para los programas dados.

## Fuente de estudio

Enunciado del TP8 en la carpeta `../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
