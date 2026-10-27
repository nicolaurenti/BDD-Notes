---
tags:
  - tp8
---

# Bloqueos y protocolo de dos fases

Un **bloqueo** reserva el acceso a un ítem. Si otra transacción pide un modo incompatible, espera: la operación siguiente todavía no se ejecuta.

## Compartido y exclusivo

- `r-lock(X)` o **S**: compartido, permite leer.
- `w-lock(X)` o **X**: exclusivo, permite escribir y también leer.
- `unlock(X)`: libera el bloqueo.

| Bloqueo concedido a otra transacción | Solicita S | Solicita X |
|---|---|---|
| S | Compatible | Espera |
| X | Espera | Espera |

Varios lectores pueden compartir un ítem. Un escritor requiere exclusividad. El bloqueo binario, más simple, solo distingue bloqueado/libre. La **granularidad** es el tamaño protegido: fila, página o tabla; un bloqueo amplio reserva también datos que quizá no se usarán.

En el ejercicio 3, `s-lock(A)` se interpreta como bloqueo compartido, igual que `r-lock(A)`; el enunciado mezcla ambas notaciones.

## Por qué bloquear no alcanza

Supongamos que una transferencia libera B después de descontar y luego bloquea A para acreditar. Entre ambos pasos otra transacción puede leer el total incompleto. Cada acceso usó un bloqueo, pero no se protegió la unidad lógica completa.

## Protocolo de dos fases: P2F o 2PL

Cada transacción atraviesa dos etapas:

1. **Crecimiento:** adquiere bloqueos sin liberar ninguno.
2. **Decrecimiento:** libera bloqueos sin adquirir nuevos.

**Después del primer `unlock`, no puede pedir otro bloqueo.** Ese es el criterio rápido para revisar la grilla. El último bloqueo adquirido marca el punto de bloqueo.

`lock(A), lock(B), unlock(A), unlock(B)` cumple. `lock(A), unlock(A), lock(B)` incumple. Si todas las transacciones cumplen 2PL y respetan los modos de acceso, se garantiza serializabilidad por conflictos.

El 2PL básico puede liberar antes del commit y permitir rollback en cascada. En **2PL estricto**, los bloqueos exclusivos se mantienen hasta commit/abort; si se retienen también los compartidos hasta el final, se habla de la variante rigurosa.

## Deadlock e inanición

**Deadlock:** T1 retiene X y espera Y; T2 retiene Y y espera X. Ninguna puede continuar. Un ciclo en el grafo de **espera** permite detectarlo; el SGBD puede abortar una víctima para liberar recursos. Este grafo representa esperas actuales y es distinto del grafo de precedencia.

2PL **no evita** deadlocks. Pedir los recursos siguiendo un orden global ayuda a prevenirlos. El **2PL conservativo** reserva todo lo necesario antes de empezar; si no puede obtenerlo como conjunto, espera sin retener una parte. Evita la espera circular, pero requiere conocer los accesos y reduce concurrencia.

**Inanición:** una transacción espera o se reinicia indefinidamente mientras otras avanzan. Una política de atención justa busca evitarla; no exige que exista un ciclo de espera.

## Cómo simular el ejercicio

Llevá una tabla de bloqueos concedidos, pedidos pendientes y valores locales. Si un pedido no se concede, esa transacción se detiene ahí. Mover todos los `unlock` al final puede crear un deadlock: no corresponde continuar leyendo como si los bloqueos hubieran sido concedidos. Si hay aborto y reinicio, hay que indicarlos antes de calcular el resultado.
