---
tags:
  - tp8
---

# TP8 - Concurrencia y recuperación

**La concurrencia coordina transacciones que se ejecutan al mismo tiempo; la recuperación conserva los efectos confirmados y elimina los incompletos ante una falla.** Ambas permiten mantener una base consistente.

## Antes de empezar: transacciones y ACID

Una **transacción** es una unidad lógica de trabajo. Transferir dinero requiere descontar de una cuenta y sumar en otra: ambas operaciones forman una sola transacción.

- `BEGIN` o `START TRANSACTION`: inicia la transacción.
- `READ(X)` y `WRITE(X)`: leen y escriben un ítem. En las grillas son operaciones conceptuales, no sentencias SQL para copiar.
- `COMMIT`: confirma el trabajo.
- `ROLLBACK`: revierte los cambios de la transacción que no se confirmó.

Las propiedades **ACID** explican qué se busca:

| Propiedad | Significado en una transferencia |
|---|---|
| Atomicidad | Se realizan ambos movimientos o ninguno. |
| Consistencia | Se respetan las reglas del negocio y las restricciones. |
| Aislamiento | La interacción con otras transacciones respeta las garantías del nivel elegido. |
| Durabilidad | Una confirmación persiste aunque luego falle el sistema. |

Una transacción pasa de **activa** a **parcialmente confirmada** al terminar sus operaciones; solo queda **confirmada** cuando la confirmación es durable. Si falla, pasa a **fallida** y luego a **abortada** al revertirse; después puede reiniciarse o finalizar. Terminar la última sentencia no equivale por sí solo a confirmar.

## Parte 1: control de concurrencia

1. [[Problemas de concurrencia entre transacciones|Problemas de concurrencia]] — Cómo reconocer actualización perdida, lectura sucia y lecturas inconsistentes.
2. [[Planes serializables y grafos de precedencia|Planes y serializabilidad]] — Cómo decidir si una intercalación equivale a una ejecución serial.
3. [[Bloqueos y protocolo de dos fases|Bloqueos y dos fases]] — Qué accesos esperan, por qué bloquear no alcanza y cuándo aparece un deadlock.
4. [[Concurrencia y aislamiento en PostgreSQL|Aislamiento en PostgreSQL]] — MVCC, niveles de aislamiento y diferencias respecto del modelo teórico.

## Parte 2: recuperación

5. [[Log de transacciones y estrategias de recuperación|Log y estrategias de recuperación]] — Qué guardan los registros y cuándo corresponde UNDO o REDO.
6. [[Checkpoints y análisis de fallas|Checkpoints y análisis de fallas]] — Cómo clasificar transacciones y reconstruir los datos sin omitir operaciones anteriores al checkpoint.

## Cómo usarlo con el práctico

Concurrencia: ejercicio 1 → problemas; 2 → grafos; 3 → bloqueos; 4 → PostgreSQL. Recuperación: ejercicio 1 → checkpoints; 2 → comparación de estrategias; 3 → recorrido del log y valores finales.

Los ejemplos son ilustrativos y no ejecutan SQL. El apunte explica el método; no reemplaza una resolución completa de las grillas del TP.

## Material

Rutas dentro de la carpeta Bases de Datos, fuera del baúl:

- Enunciado: `Practicos/Practico 8 - Concurrencia y Recuperacion.docx`.
- Filminas: `Contenido extra/Filminas y Practicos de cursada/2023/BDI_08 (Proc_Transacciones).pdf`, especialmente páginas 2–29.
- Los detalles propios de PostgreSQL tienen referencias oficiales en su subtema.
