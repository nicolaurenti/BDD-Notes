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

## Derivar cada necesidad con una pregunta

Primero preguntar: **¿pudo llegar al disco un cambio no confirmado?** Si sí, una transacción abortada puede dejar datos que deben deshacerse: se necesita UNDO. Después preguntar: **¿puede existir un COMMIT sin todas sus páginas de datos persistidas?** Si sí, sus efectos pueden necesitar REDO.

En actualización diferida, antes del COMMIT los efectos se conservan en log y espacio de trabajo, sin aplicarlos a la base persistida. Si T falla antes de confirmar, se descartan sin deshacer páginas. Si confirma pero la falla interrumpe la aplicación, el log permite REDO: NO UNDO/REDO.

En inmediata, pueden persistir páginas antes del COMMIT, de modo que se necesita UNDO de las no confirmadas. Si al confirmar se fuerzan todas las páginas, no hace falta REDO de una confirmada: UNDO/NO REDO. Si no se fuerzan, también puede haber páginas pendientes de confirmadas: UNDO/REDO.

Ejemplo: T1 cambia A de 10 a 15 y confirma; T2 cambia B de 20 a 30 y no confirma antes de la falla. En UNDO/REDO, T1 puede requerir rehacer A=15 y T2 deshacer B=20. En diferida no se restaura B porque el 30 no debía haber llegado a la base; en inmediata con FORCE, el A=15 ya debía estar persistido al confirmar.

## Paginación de sombra en el material

Las filminas también describen conservar un directorio de páginas anterior y escribir los cambios en páginas nuevas. Si la transacción falla, se vuelve al directorio anterior; si confirma, se consolida el nuevo. En ese modelo la recuperación es NO UNDO/NO REDO. Su presencia en el tema no cambia la estrategia pedida por un ejercicio de log: primero identificar cuál declara la consigna.

## Fuente de estudio

Enunciado del TP8 en la carpeta `../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
