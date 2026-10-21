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

[[TP8 - Ejemplo de transferencia y propiedades ACID|Ejemplo de transferencia y propiedades ACID]]

## Estados y terminación

Una transacción activa puede alcanzar COMMIT o terminar por ABORT/ROLLBACK. Confirmar significa aceptar todos sus efectos como una unidad; abortar exige revertir los que haya producido. En los ejercicios de recuperación, la presencia de START sin COMMIT en el prefijo disponible indica que no llegó a confirmarse antes de la falla. No usar el COMMIT que figura después del punto de corte.

## Fuente de estudio

Enunciado del TP8 en la carpeta `../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
