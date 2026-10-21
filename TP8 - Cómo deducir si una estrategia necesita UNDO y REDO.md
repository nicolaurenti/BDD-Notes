---
tags:
  - tp8
---

# TP8 - Cómo deducir si una estrategia necesita UNDO y REDO

Razonamiento paso a paso de diferida, inmediata con FORCE e inmediata sin FORCE, con un caso de dos transacciones.

## Derivar cada necesidad con una pregunta

Primero preguntar: **¿pudo llegar al disco un cambio no confirmado?** Si sí, una transacción abortada puede dejar datos que deben deshacerse: se necesita UNDO. Después preguntar: **¿puede existir un COMMIT sin todas sus páginas de datos persistidas?** Si sí, sus efectos pueden necesitar REDO.

En actualización diferida, antes del COMMIT los efectos se conservan en log y espacio de trabajo, sin aplicarlos a la base persistida. Si T falla antes de confirmar, se descartan sin deshacer páginas. Si confirma pero la falla interrumpe la aplicación, el log permite REDO: NO UNDO/REDO.

En inmediata, pueden persistir páginas antes del COMMIT, de modo que se necesita UNDO de las no confirmadas. Si al confirmar se fuerzan todas las páginas, no hace falta REDO de una confirmada: UNDO/NO REDO. Si no se fuerzan, también puede haber páginas pendientes de confirmadas: UNDO/REDO.

Ejemplo: T1 cambia A de 10 a 15 y confirma; T2 cambia B de 20 a 30 y no confirma antes de la falla. En UNDO/REDO, T1 puede requerir rehacer A=15 y T2 deshacer B=20. En diferida no se restaura B porque el 30 no debía haber llegado a la base; en inmediata con FORCE, el A=15 ya debía estar persistido al confirmar.

## Fuente de estudio

Enunciado del TP8 en la carpeta `../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
