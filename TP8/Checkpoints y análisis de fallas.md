---
tags:
  - tp8
---

# Checkpoints y análisis de fallas

Un **checkpoint** o punto de revisión reduce cuánto historial debe examinarse para recuperar. **No confirma las transacciones activas ni reemplaza un backup.**

## Supuesto usado por la cátedra

Las filminas presentan un checkpoint simplificado: se suspende brevemente la ejecución, se fuerzan los cambios de transacciones confirmadas, se persiste el registro de checkpoint y se continúa.

Bajo ese supuesto, lo confirmado antes del checkpoint ya está en disco. Una transacción que lo atraviesa sigue activa: sus registros anteriores pueden ser necesarios. Otros sistemas usan checkpoints difusos con metadatos y reglas distintas; no conviene trasladar esta simplificación sin revisar el enunciado.

## Qué transacciones considerar

Para una falla bajo **UNDO/REDO**, suponiendo que no hay abortos pendientes de completar:

| Situación | Acción |
|---|---|
| Confirmó antes del checkpoint | Ninguna: sus cambios ya se garantizaron en disco. |
| Comenzó antes del checkpoint y confirmó después, antes de la falla | REDO de sus cambios necesarios, incluidos los anteriores al checkpoint. |
| Comenzó después del checkpoint y confirmó antes de la falla | REDO. |
| Seguía activa al fallar, haya empezado antes o después del checkpoint | UNDO de sus escrituras que pudieran haber llegado al disco. |

Con **NO UNDO/REDO**, las activas no requieren UNDO. Con **UNDO/NO REDO**, las confirmadas no requieren REDO. Siempre clasificá primero según la estrategia.

## Ejemplo de recorrido

```text
<T1, START>
<T1, A, 10, 15>
<CHECKPOINT>
<T1, COMMIT>
<T2, START>
<T2, B, 20, 25>
FALLA
```

En UNDO/REDO, T1 confirmó después del checkpoint: **REDO**, A=15. T2 no confirmó: **UNDO**, B=20. El resultado es A=15 y B=20.

Si se mirara únicamente el fragmento posterior al checkpoint, se perdería la escritura de T1. Si la estrategia fuera UNDO/NO REDO, T1 ya tendría A=15 garantizado al confirmar; si fuera diferida, B nunca habría sido modificado en disco por T2.

## Método para resolver los ejercicios

1. **Cortá el log en el punto de falla solicitado.** En el ejercicio 3 cada inciso es un corte independiente; no uses commits posteriores al corte.
2. Anotá por transacción START, COMMIT/ABORT y posición respecto del checkpoint.
3. Elegí conjuntos UNDO/REDO según la política. Una transacción sin commit durable no se trata como confirmada.
4. Recuperá sus registros relevantes, incluso anteriores al checkpoint. Para una línea temporal, marcá inicio, confirmación, checkpoint y falla.
5. Seguí las escrituras a deshacer hacia atrás y las que deban rehacerse hacia adelante, respetando el algoritmo del modelo. En los ejemplos sin dependencias conflictivas entre ambos conjuntos, sus efectos se reconstruyen directamente. Con escrituras cruzadas o lecturas sucias hace falta analizar también la recuperabilidad; no alcanza con aplicar dos listas arbitrariamente.
6. Registrá los valores finales en una tabla, justificando qué escritura válida queda sobre cada ítem. Si no se conoce un valor inicial y ningún registro permite deducirlo, indicá que falta información.

## Advertencia concreta del ejercicio 3

El log tiene valores previos incompatibles con la secuencia si se lo interpreta como un historial completo sobre un único valor por ítem: T1 deja D3=35 y luego T3 registra D3: **0→50**. T2 deja D4=0, pero después aparecen escrituras de D4 con valor anterior **70** y luego **90** sin operaciones intermedias que los expliquen.

No corrijas esos números en silencio. La pertenencia a UNDO/REDO se obtiene de los commits visibles; para un estado final numérico hay que declarar si se aplican literalmente los registros o qué corrección/supuesto se adopta. El log, tal como está, no representa un historial completo coherente del modelo simple.

## Checkpoint y backup

El checkpoint guarda avance para acelerar recuperación con el almacenamiento disponible. Un **backup** conserva una copia recuperable ante pérdida de archivos: completo si copia toda la base, incremental si conserva cambios respecto de un respaldo previo. Son mecanismos complementarios.
