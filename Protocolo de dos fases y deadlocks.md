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

Declarar la convención usada si la cátedra utiliza otra terminología. El 2PL básico puede liberar una escritura antes del COMMIT y permitir dependencias sobre datos no confirmados.

### Ejercicio 3: A=100, B=200

T1 transfiere 50 de B a A. En cualquiera de los dos órdenes seriales, T2 visualiza 300.

En el schedule S dado, T1 baja B a 150 y libera B; T2 lee A=100 y B=150 antes de que T1 suba A a 150: visualiza **250**. T1 libera B y luego adquiere A; T2 libera A y luego adquiere B. No siguen 2PL.

Si ambas conservan todos sus bloqueos hasta terminar, ya no se puede ejecutar ese mismo intercalado sin esperas. En el prefijo que deja B en manos de T1 y A en manos de T2, T2 espera B y T1 espera A: **deadlock**. El motor debe abortar una transacción y reintentarla, o elegir otra planificación. Las ejecuciones completadas serializables permiten el total 300; no ignorar el bloqueo mutuo para forzar una respuesta numérica.
