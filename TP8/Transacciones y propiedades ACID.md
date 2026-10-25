---
tags:
  - tp8
---

# Transacciones y propiedades ACID

Una transacción agrupa operaciones que deben tratarse como una unidad.

| Propiedad | Significado |
|---|---|
| Atomicidad | O se completan sus efectos o se revierten |
| Consistencia | Las reglas de integridad se mantienen entre estados válidos |
| Aislamiento | La interacción concurrente respeta el nivel acordado |
| Durabilidad | Los efectos confirmados sobreviven a fallos según las garantías del sistema |

```sql
BEGIN;
-- Operaciones de la unidad de trabajo.
COMMIT;
```

`ROLLBACK` cancela la transacción. Atomicidad no evita por sí sola interferencias de otras transacciones.

## Ejemplo de transferencia

La transferencia permite identificar la unidad de trabajo y qué asegura cada propiedad. Las dos cuentas deben reflejar la misma transferencia completa.

### Ejemplo de transferencia y propiedades ACID

Dos saldos iniciales y una transferencia de 5000 para distinguir atomicidad, consistencia, aislamiento y durabilidad.

T1 debe restar 5000 al cliente 10 del banco 1 y sumarlos al cliente 13 del banco 2. Con saldos iniciales 10000 y 30000, el estado final correcto es 5000 y 35000: la suma sigue siendo 40000.

Atomicidad exige que no quede confirmado solo el débito. Consistencia exige respetar las reglas de la base antes y después de la unidad de trabajo. Aislamiento evita que una operación concurrente obtenga resultados incompatibles con el modelo de ejecución admitido. Durabilidad asegura la persistencia de la transferencia confirmada frente a una falla contemplada por el sistema.

```sql
BEGIN;
UPDATE cliente
SET saldo = saldo - 5000
WHERE id_cliente = 10 AND id_banco = 1;

UPDATE cliente
SET saldo = saldo + 5000
WHERE id_cliente = 13 AND id_banco = 2;
COMMIT;
```

Es una representación de la unidad lógica, no una ejecución sobre la base. En una implementación completa también deben comprobarse existencia de las cuentas y reglas de saldo que corresponda exigir. BEGIN y COMMIT no convierten una lógica incorrecta en correcta.

## Estados y terminación

Una transacción activa puede alcanzar COMMIT o terminar por ABORT/ROLLBACK. Confirmar significa aceptar todos sus efectos como una unidad; abortar exige revertir los que haya producido. En los ejercicios de recuperación, la presencia de START sin COMMIT en el prefijo disponible indica que no llegó a confirmarse antes de la falla. No usar el COMMIT que figura después del punto de corte.

## Fuente de estudio

Enunciado del TP8 en la carpeta `../../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
