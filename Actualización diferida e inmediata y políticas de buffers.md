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

## Deducir la recuperación requerida

Para deducir UNDO se pregunta si pudieron persistir escrituras sin COMMIT. Para deducir REDO se pregunta si pudo confirmarse sin persistir todas las páginas de datos.

[[TP8 - Cómo deducir si una estrategia necesita UNDO y REDO|Cómo deducir si una estrategia necesita UNDO y REDO]]

## Paginación de sombra en el material

Las filminas también describen conservar un directorio de páginas anterior y escribir los cambios en páginas nuevas. Si la transacción falla, se vuelve al directorio anterior; si confirma, se consolida el nuevo. En ese modelo la recuperación es NO UNDO/NO REDO. Su presencia en el tema no cambia la estrategia pedida por un ejercicio de log: primero identificar cuál declara la consigna.

## Fuente de estudio

Enunciado del TP8 en la carpeta `../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
