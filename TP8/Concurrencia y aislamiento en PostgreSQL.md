---
tags:
  - tp8
---

# Concurrencia y aislamiento en PostgreSQL

## MVCC: versiones visibles

PostgreSQL usa **MVCC**: una lectura común consulta versiones según una instantánea o *snapshot*, sin esperar normalmente a los escritores. También utiliza bloqueos; MVCC no elimina los conflictos entre escrituras ni las esperas por operaciones incompatibles. [Referencia oficial](https://www.postgresql.org/docs/current/mvcc-intro.html).

La duración de la instantánea depende del aislamiento. Siempre se pueden ver los cambios previos de la propia transacción.

## Niveles de aislamiento

| Nivel solicitado | Comportamiento en PostgreSQL |
|---|---|
| READ UNCOMMITTED | Se acepta, pero se comporta como READ COMMITTED. |
| READ COMMITTED | Predeterminado. Cada sentencia ve una instantánea nueva de datos confirmados. |
| REPEATABLE READ | Mantiene la instantánea desde la primera sentencia que requiere una. No admite lecturas no repetibles ni fantasmas; puede haber anomalías de serialización. |
| SERIALIZABLE | Agrega detección de dependencias peligrosas; puede abortar una transacción. Hay que reintentarla completa. |

No hay lecturas sucias en ninguno. En READ COMMITTED, dos consultas sucesivas pueden diferir. En el estándar, REPEATABLE READ permite fantasmas; PostgreSQL ofrece una garantía mayor. [Documentación de aislamiento](https://www.postgresql.org/docs/current/transaction-iso.html).

**Ejemplo:** T1 consulta un saldo; T2 lo modifica y confirma; T1 repite la consulta. En READ COMMITTED puede ver el cambio. En REPEATABLE READ sigue viendo su instantánea, si no lo modificó ella misma.

## Elegir aislamiento y bloquear explícitamente

Ejemplos informativos, sin ejecutarlos:

```sql
BEGIN ISOLATION LEVEL SERIALIZABLE;
-- operaciones de una unidad lógica
COMMIT;
```

Para leer una fila con intención de modificarla:

```sql
BEGIN;
SELECT saldo FROM cliente
WHERE id_cliente = 10 AND id_banco = 1
FOR UPDATE;
-- calcular y actualizar dentro de esta misma transacción
COMMIT;
```

`FOR UPDATE` bloquea las filas seleccionadas frente a escritores o pedidos incompatibles hasta el fin de la transacción; una lectura común puede seguir viendo su versión visible. Las esperas pueden producir deadlocks, que PostgreSQL detecta abortando una transacción. [Bloqueos explícitos](https://www.postgresql.org/docs/current/explicit-locking.html).

## Dos matices de las filminas

“No dispone de READ UNCOMMITTED” significa que **no tiene un comportamiento independiente** para ese nivel; el nombre sí se acepta.

La afirmación “todos los niveles impiden actualización perdida” no debe aplicarse a cualquier secuencia de la aplicación. Dos sesiones pueden leer 100 y luego enviar valores absolutos 120 y 90; si la segunda escritura usa una lectura vieja, puede sobrescribir el cambio previo en READ COMMITTED. Una modificación relativa como `SET saldo = saldo + 20`, o proteger lectura y actualización con el bloqueo adecuado, evita ese patrón. El tratamiento de actualizaciones concurrentes se explica en la [sección READ COMMITTED](https://www.postgresql.org/docs/current/transaction-iso.html#XACT-READ-COMMITTED).

**Qué recordar:** instantánea estable no equivale a serializabilidad; una operación de negocio debe conservar su unidad transaccional y manejar los reintentos cuando corresponda.
