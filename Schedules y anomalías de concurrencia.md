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

### Ejemplo del banco: actualización perdida

Saldo inicial del cliente 10: 10000. T1 prepara una resta de 5000 y T3 un depósito de 15000.

```text
T1 lee 10000 y calcula 5000.
T3 lee 10000 y calcula 25000.
T3 escribe 25000 y confirma.
T1 escribe su cálculo viejo: 5000 y confirma.
```

Se pierde el depósito: el saldo correcto al aplicar ambas operaciones sería 20000. Es una simulación de un sistema sin control adecuado o de una aplicación que escribe resultados viejos; no afirmar que cualquier UPDATE aritmético en PostgreSQL se comporta así.

### Lectura sucia y transferencia parcial

Si T1 escribe el débito, T2 lo lee antes del COMMIT y después T1 aborta, T2 usó un valor que nunca quedó confirmado. Si T2 observa el débito sin el crédito correspondiente, puede obtener un total 5000 menor al correcto. Son problemas relacionados pero distintos.
