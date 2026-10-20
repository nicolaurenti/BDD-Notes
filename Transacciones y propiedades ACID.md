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
