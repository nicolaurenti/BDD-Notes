---
tags:
  - tp8
---

# TP8 - Ejemplo temporal de checkpoint y falla

Línea de tiempo bajo el modelo de checkpoint de la cátedra y la estrategia UNDO/REDO.

## Ejemplo temporal

```text
T1: START -- escritura -- |CHECKPOINT| -- COMMIT -- |FALLA|
T2: START -- COMMIT ----- |CHECKPOINT| ------------ |FALLA|
T3:                      |CHECKPOINT| -- START --- |FALLA|
```

T2 quedó asegurada por el checkpoint. T1 confirma después y se considera para REDO en UNDO/REDO, incluyendo las escrituras suyas que no quedaron aseguradas por el control. T3 no confirma y se considera para UNDO si pudo escribir páginas.

Para dibujar el gráfico, marcar START y COMMIT de cada transacción sobre una línea de tiempo común, y luego las dos verticales de checkpoint y falla. Una línea abierta en la falla no se transforma en confirmada porque haya comenzado hace mucho.

## Fuente de estudio

Enunciado del TP8 en la carpeta `../../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
