---
tags:
  - tp8
---

# Protocolo de dos fases y deadlocks


1. **Crecimiento:** adquirir bloqueos; no liberar.
2. **Decrecimiento:** liberar; no adquirir otros nuevos.

No significa «dos bloqueos» ni «dos transacciones». Garantiza serializabilidad por conflictos para el modelo de bloqueos utilizado, pero puede provocar deadlocks.

- **2PL estricto:** retiene bloqueos exclusivos hasta COMMIT/ABORT.
- **2PL riguroso:** retiene compartidos y exclusivos hasta el final.

En el TP, el criterio básico a verificar es que no se adquieran nuevos bloqueos después del primer unlock. El 2PL básico puede liberar una escritura antes del COMMIT y permitir dependencias sobre datos no confirmados.

## Aplicación al caso de A y B

La simulación distingue el intercalado original y el que conserva los bloqueos hasta el final. Posponer los unlock puede introducir esperas que impiden continuar ese mismo orden.

[[TP8 - Simulación de bloqueos del ejercicio 3|Simulación de bloqueos del ejercicio 3]]

## Deadlock, inanición y dos fases conservativo

Deadlock significa espera circular: ninguna de las transacciones involucradas puede avanzar por sí sola. El sistema puede detectarlo con el grafo de esperas y abortar una víctima para liberar recursos. Inanición o livelock significa que una transacción no progresa durante un tiempo indefinido mientras otras sí avanzan; no exige un ciclo fijo.

Las filminas presentan la variante conservativa: obtener todos los bloqueos requeridos antes de comenzar. Si alguno no se puede obtener, se espera sin retener un conjunto parcial que produzca el ciclo. Evita ese deadlock a cambio de restringir la concurrencia y requerir conocer los datos que se van a usar. Dos fases básico garantiza serializabilidad por conflictos, pero no garantiza ausencia de deadlock.

## Fuente de estudio

Enunciado del TP8 en la carpeta `../../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
