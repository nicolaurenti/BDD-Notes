---
tags:
  - tp8
---

# TP8 - Simulación de bloqueos del ejercicio 3

Caso A=100, B=200: suma observada, violaciones de dos fases y espera circular al conservar los bloqueos.

## Ejercicio 3: A=100, B=200

T1 transfiere 50 de B a A. En cualquiera de los dos órdenes seriales, T2 visualiza 300.

En el schedule S dado, T1 baja B a 150 y libera B; T2 lee A=100 y B=150 antes de que T1 suba A a 150: visualiza **250**. T1 libera B y luego adquiere A; T2 libera A y luego adquiere B. No siguen 2PL.

Si ambas conservan todos sus bloqueos hasta terminar, ya no se puede ejecutar ese mismo intercalado sin esperas. En el prefijo que deja B en manos de T1 y A en manos de T2, T2 espera B y T1 espera A: **deadlock**. El motor debe abortar una transacción y reintentarla, o elegir otra planificación. Las ejecuciones completadas serializables permiten el total 300; no ignorar el bloqueo mutuo para forzar una respuesta numérica.

## Reconocer la infracción sin calcular los saldos

En T1 del ejercicio 3 aparece `unlock(B)` antes de `w-lock(A)`. Con eso basta para demostrar que no sigue dos fases: después de comenzar a liberar, intenta adquirir otro bloqueo. En T2 ocurre lo mismo entre unlock(A) y r-lock(B). No se necesita que haya un resultado incorrecto en todas las ejecuciones para demostrar la infracción del protocolo.

Si todos los unlock se trasladan al final, hay crecimiento seguido de decrecimiento. Pero conservar el prefijo que entrega B a T1 y A a T2 produce espera circular: T1 necesita A y T2 necesita B. En ese punto no hay un valor final visualizado; primero se debe resolver la espera mutua. Al ejecutar completamente una planificación serializable, el total que representa la transferencia es 300.

## Fuente de estudio

Enunciado del TP8 en la carpeta `../../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
