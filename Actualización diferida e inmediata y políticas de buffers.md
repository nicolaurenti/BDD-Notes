---
tags:
  - tp8
---

# Actualización diferida e inmediata y políticas de buffers


| Política | Qué permite o exige | Consecuencia de recuperación |
|---|---|---|
| STEAL | Escribir páginas con cambios no confirmados | Puede requerir UNDO |
| NO-STEAL | No persistir cambios no confirmados | No necesita UNDO de esos cambios |
| FORCE | Asegurar datos de la transacción al confirmar | No requiere REDO por páginas confirmadas pendientes |
| NO-FORCE | Confirmar sin forzar todas las páginas de datos | Puede requerir REDO |

Combinaciones del modelo de estudio:

| Estrategia | Política típica | Recuperación |
|---|---|---|
| Diferida | NO-STEAL + NO-FORCE | NO UNDO / REDO |
| Inmediata | STEAL + FORCE | UNDO / NO REDO |
| Inmediata | STEAL + NO-FORCE | UNDO / REDO |

NO-STEAL + FORCE evita ambas necesidades en este modelo. «Inmediata» no determina por sí sola si hay REDO: hay que conocer FORCE/NO-FORCE.

PostgreSQL usa WAL y MVCC; no trasladar literalmente el esquema didáctico de restaurar cada valor anterior al mecanismo real de crash recovery del motor.
