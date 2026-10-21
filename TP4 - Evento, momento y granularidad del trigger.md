---
tags:
  - tp4
---

# TP4 - Evento, momento y granularidad del trigger

Criterios para elegir BEFORE o AFTER, ROW o STATEMENT y ubicar las ejecuciones.

## Evento, momento y granularidad

El **evento** puede ser INSERT, UPDATE o DELETE. El **momento** indica si el cuerpo actúa antes o después de la modificación. La **granularidad** indica cuántas veces se ejecuta.

| Decisión | Cuándo elegirla en estos ejercicios |
|---|---|
| BEFORE ROW | Revisar la fila propuesta o corregir NEW antes de guardarla |
| AFTER ROW | Registrar una modificación realizada o mantener otro dato a partir de esa fila |
| BEFORE STATEMENT | Preparar una acción para toda la sentencia |
| AFTER STATEMENT | Registrar o comprobar el efecto global de la sentencia |

Si un UPDATE afecta tres filas, ROW ejecuta el cuerpo tres veces y STATEMENT una. Si afecta cero, ROW no tiene filas para ejecutar el cuerpo, pero STATEMENT sí se dispara. Por eso «una ejecución del trigger» no significa siempre «una fila modificada».

Para simular, ubicar primero BEFORE STATEMENT; luego las actuaciones por cada fila, considerando los BEFORE ROW antes de la operación de esa fila; y las acciones AFTER correspondientes. AFTER significa posterior a la modificación, no una nueva sentencia del usuario ni una confirmación independiente.

## Fuente de estudio

Enunciado del TP4 en la carpeta `../Practicos` y filminas de cátedra `BDI_04 (SQL Procedural).pdf` (carpeta de cursada 2023).
