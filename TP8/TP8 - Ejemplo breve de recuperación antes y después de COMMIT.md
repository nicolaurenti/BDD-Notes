---
tags:
  - tp8
---

# TP8 - Ejemplo breve de recuperación antes y después de COMMIT

Log pequeño de T1 sobre A y T2 sobre B, con dos cortes distintos.

## Resolver cada corte como un log distinto

No mirar el log completo y clasificar para siempre a T1 como confirmada. En el corte que termina en `<T1, D3, 50, 35>`, su COMMIT todavía no existe; en el corte final sí. Marcar START y COMMIT solo dentro del prefijo correspondiente y después construir las listas UNDO y REDO.

Ejemplo breve independiente del log impreso:

```text
<T1, START>
<T1, A, 10, 15>
<T2, START>
<T2, B, 20, 30>
<T1, COMMIT>
FALLA
```

Con UNDO/REDO: T1 está en REDO y T2 en UNDO. Deshacer T2 restaura B=20; rehacer T1 asegura A=15. Si se corta antes de `<T1, COMMIT>`, ambas están sin confirmar: se restaura A=10 y B=20. El dato decisivo fue el COMMIT disponible, no que una transacción pareciera «terminada» en un relato externo.

## Fuente de estudio

Enunciado del TP8 en la carpeta `../../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
