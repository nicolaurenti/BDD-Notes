---
tags:
  - tp8
---

# Checkpoints y transacciones activas

El checkpoint del modelo de la cátedra registra un punto en que se fuerzan a disco los efectos de las transacciones ya confirmadas. Las filminas describen suspender temporalmente el procesamiento, escribir esos efectos y el registro de control, y reanudarlo. Se usa ese modelo para resolver los gráficos y logs del TP8.

## Qué garantiza y qué no

No confirma transacciones activas. Si T1 había empezado y todavía no tenía COMMIT, sigue activa después del checkpoint. Una escritura de T1 anterior al control no queda automáticamente fuera del análisis: la decisión se toma por la situación de **T1**, no solo por la posición de esa escritura.

Para UNDO/REDO, bajo ese modelo:

| Situación | Acción |
|---|---|
| Confirmada antes del checkpoint, efectos asegurados | No necesita recuperación adicional |
| Confirmada después del checkpoint y antes de la falla | REDO de sus efectos que corresponda recuperar |
| No confirmada al fallar, con posibilidad de páginas escritas | UNDO |

La estrategia puede quitar alguna necesidad: en diferida no se hace UNDO de datos persistidos no confirmados; con FORCE al COMMIT no se necesita REDO de confirmadas.

## Ejemplo temporal

```text
T1: START -- escritura -- |CHECKPOINT| -- COMMIT -- |FALLA|
T2: START -- COMMIT ----- |CHECKPOINT| ------------ |FALLA|
T3:                      |CHECKPOINT| -- START --- |FALLA|
```

T2 quedó asegurada por el checkpoint. T1 confirma después y se considera para REDO en UNDO/REDO, incluyendo las escrituras suyas que no quedaron aseguradas por el control. T3 no confirma y se considera para UNDO si pudo escribir páginas.

Para dibujar el gráfico, marcar START y COMMIT de cada transacción sobre una línea de tiempo común, y luego las dos verticales de checkpoint y falla. Una línea abierta en la falla no se transforma en confirmada porque haya comenzado hace mucho.

## Fuente de estudio

Enunciado del TP8 en la carpeta `../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
