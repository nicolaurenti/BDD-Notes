---
tags:
  - tp8
---

# Bloqueos, protocolo de dos fases y deadlocks

| Bloqueos sobre el mismo ítem | Compatible |
|---|---|
| S con S | Sí |
| S con X | No |
| X con X | No |

S permite leer; X permite escribir. En el práctico aparecen `r-lock` y `s-lock` como bloqueo compartido, y `w-lock` como exclusivo.

Una solicitud incompatible espera hasta que se libere el bloqueo. Al simular, una transacción bloqueada no puede seguir ejecutando instrucciones posteriores como si hubiera adquirido el recurso.

## Lectura, escritura y espera

Antes de leer un ítem se solicita el compartido; antes de escribir, el exclusivo. Si T1 tiene S(A), T2 puede obtener S(A), pero no X(A). Si T1 tiene X(A), ninguna de esas solicitudes de T2 se concede hasta liberar el bloqueo.

En el TP se usa tanto r-lock como s-lock para compartido: tomar el significado declarado por el enunciado, sin interpretar la letra s como «exclusivo». W-lock corresponde a escritura.

Supongamos T1 con X(B) y T2 que solicita S(B). Se anota «T2 espera por T1». Mientras espera, T2 no puede ejecutar su lectura de B, liberar B como si lo tuviera y calcular A+B. Primero debe obtenerlo. Esta regla cambia el resultado de una simulación si se trasladan los unlock al final.

## Diferenciar solicitud y concesión

En una tabla temporal conviene anotar por ítem quién tiene el bloqueo y quién espera. Un comando escrito en el schedule es una solicitud; solo cuando resulta compatible se continúa con la siguiente operación de esa transacción. El grafo de esperas tiene una arista del que espera al que retiene el recurso. Se usa para deadlock y no debe confundirse con el grafo de conflictos de serializabilidad.

## Protocolo de dos fases y deadlocks

1. **Crecimiento:** adquirir bloqueos; no liberar.
2. **Decrecimiento:** liberar; no adquirir otros nuevos.

No significa «dos bloqueos» ni «dos transacciones». Garantiza serializabilidad por conflictos para el modelo de bloqueos utilizado, pero puede provocar deadlocks.

- **2PL estricto:** retiene bloqueos exclusivos hasta COMMIT/ABORT.
- **2PL riguroso:** retiene compartidos y exclusivos hasta el final.

En el TP, el criterio básico a verificar es que no se adquieran nuevos bloqueos después del primer unlock. El 2PL básico puede liberar una escritura antes del COMMIT y permitir dependencias sobre datos no confirmados.

### Aplicación al caso de A y B

La simulación distingue el intercalado original y el que conserva los bloqueos hasta el final. Posponer los unlock puede introducir esperas que impiden continuar ese mismo orden.

#### Simulación de bloqueos del ejercicio 3

Caso A=100, B=200: suma observada, violaciones de dos fases y espera circular al conservar los bloqueos.

#### Ejercicio 3: A=100, B=200

T1 transfiere 50 de B a A. En cualquiera de los dos órdenes seriales, T2 visualiza 300.

En el schedule S dado, T1 baja B a 150 y libera B; T2 lee A=100 y B=150 antes de que T1 suba A a 150: visualiza **250**. T1 libera B y luego adquiere A; T2 libera A y luego adquiere B. No siguen 2PL.

Si ambas conservan todos sus bloqueos hasta terminar, ya no se puede ejecutar ese mismo intercalado sin esperas. En el prefijo que deja B en manos de T1 y A en manos de T2, T2 espera B y T1 espera A: **deadlock**. El motor debe abortar una transacción y reintentarla, o elegir otra planificación. Las ejecuciones completadas serializables permiten el total 300; no ignorar el bloqueo mutuo para forzar una respuesta numérica.

#### Reconocer la infracción sin calcular los saldos

En T1 del ejercicio 3 aparece `unlock(B)` antes de `w-lock(A)`. Con eso basta para demostrar que no sigue dos fases: después de comenzar a liberar, intenta adquirir otro bloqueo. En T2 ocurre lo mismo entre unlock(A) y r-lock(B). No se necesita que haya un resultado incorrecto en todas las ejecuciones para demostrar la infracción del protocolo.

Si todos los unlock se trasladan al final, hay crecimiento seguido de decrecimiento. Pero conservar el prefijo que entrega B a T1 y A a T2 produce espera circular: T1 necesita A y T2 necesita B. En ese punto no hay un valor final visualizado; primero se debe resolver la espera mutua. Al ejecutar completamente una planificación serializable, el total que representa la transferencia es 300.

### Deadlock, inanición y dos fases conservativo

Deadlock significa espera circular: ninguna de las transacciones involucradas puede avanzar por sí sola. El sistema puede detectarlo con el grafo de esperas y abortar una víctima para liberar recursos. Inanición o livelock significa que una transacción no progresa durante un tiempo indefinido mientras otras sí avanzan; no exige un ciclo fijo.

Las filminas presentan la variante conservativa: obtener todos los bloqueos requeridos antes de comenzar. Si alguno no se puede obtener, se espera sin retener un conjunto parcial que produzca el ciclo. Evita ese deadlock a cambio de restringir la concurrencia y requerir conocer los datos que se van a usar. Dos fases básico garantiza serializabilidad por conflictos, pero no garantiza ausencia de deadlock.

## Fuente de estudio

Enunciado del TP8 en la carpeta `../../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
