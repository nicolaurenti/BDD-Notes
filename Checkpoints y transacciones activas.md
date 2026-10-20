---
tags:
  - tp8
---

# Checkpoints y transacciones activas


Un checkpoint limita trabajo de recuperación según el algoritmo: no es un COMMIT de todas las transacciones. Una transacción activa durante el checkpoint puede necesitar atención después de la falla.

En un modelo didáctico donde el checkpoint deja durables los efectos ya confirmados antes de él:

| Situación en la falla | UNDO/REDO habitual |
|---|---|
| Confirmó antes del checkpoint y sus efectos están asegurados | Ninguna acción adicional |
| Confirmó después y sus páginas pueden estar pendientes | REDO |
| No confirmó y pudo persistir escrituras | UNDO |

En checkpoints no quiescentes o difusos, el punto de arranque y las transacciones pendientes se obtienen de metadatos del algoritmo. No usar la regla «ignorar todo lo anterior al checkpoint» sin esa garantía. [Checkpoints y WAL en PostgreSQL](https://www.postgresql.org/docs/current/wal-configuration.html).
