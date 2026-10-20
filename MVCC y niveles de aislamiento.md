---
tags:
  - tp8
---

# MVCC y niveles de aislamiento


MVCC conserva versiones de filas para que cada consulta vea una instantánea apropiada. Una lectura común no bloquea una escritura de fila de la misma forma que en el modelo elemental de locks; sí existen bloqueos por otras operaciones. [MVCC](https://www.postgresql.org/docs/current/mvcc-intro.html).

| Nivel solicitado | Comportamiento relevante en PostgreSQL |
|---|---|
| READ UNCOMMITTED | Se comporta como READ COMMITTED; no permite lecturas sucias |
| READ COMMITTED | Por defecto; una nueva sentencia puede ver nuevos commits |
| REPEATABLE READ | Instantánea estable de la transacción; PostgreSQL también evita fantasmas en sus lecturas, pero puede haber anomalías de serialización |
| SERIALIZABLE | Detecta situaciones incompatibles con una ejecución serial y puede exigir reintentar toda la transacción |

```sql
BEGIN ISOLATION LEVEL SERIALIZABLE;
-- Lecturas y escrituras de la unidad de trabajo.
COMMIT;
```

READ COMMITTED no garantiza que una secuencia arbitraria de «leer, calcular fuera y escribir un valor» sea segura. Puede ser necesario un UPDATE atómico, `SELECT ... FOR UPDATE` o un nivel superior según la regla. [Aislamiento](https://www.postgresql.org/docs/current/transaction-iso.html) y [bloqueos explícitos](https://www.postgresql.org/docs/current/explicit-locking.html).
