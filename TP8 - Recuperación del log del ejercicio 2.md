---
tags:
  - tp8
---

# TP8 - Recuperación del log del ejercicio 2

Se aplica el modelo de checkpoint de la cátedra al log que contiene T1, T2, T3 y T4.

## Aplicación al log del ejercicio 2

Orden relevante: T1 y T2 empiezan antes del checkpoint; T2 confirma antes; T1 confirma después; T3 queda sin COMMIT; T4 confirma antes de la falla.

Bajo la hipótesis del checkpoint didáctico anterior:

| Transacción | Diferida NO UNDO/REDO | Inmediata UNDO/NO REDO | Inmediata UNDO/REDO |
|---|---|---|---|
| T1 | REDO | Ninguna: FORCE al COMMIT | REDO |
| T2 | Ninguna: checkpoint | Ninguna | Ninguna: checkpoint |
| T3 | Descartar, sin UNDO de datos persistidos | UNDO | UNDO |
| T4 | REDO | Ninguna: FORCE al COMMIT | REDO |

Aunque algunas escrituras de T1 sean anteriores al checkpoint, no se excluye T1 solo por esa ubicación. En un algoritmo que registre páginas ya aseguradas, el trabajo concreto puede reducirse.

## Fuente de estudio

Enunciado del TP8 en la carpeta `../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
