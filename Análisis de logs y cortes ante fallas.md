---
tags:
  - tp8
---

# Análisis de logs y cortes ante fallas

Para cada corte se analiza únicamente el prefijo de log disponible: se identifican START y COMMIT, se forman las listas de recuperación según la estrategia y se recorren sus escrituras. La posición de una escritura aislada no basta para clasificar toda la transacción.

## Aplicación al log del ejercicio 2

La clasificación de T1–T4 cambia según se use diferida, inmediata con FORCE o UNDO/REDO. El desarrollo compara esas alternativas.

[[TP8 - Recuperación del log del ejercicio 2|Recuperación del log del ejercicio 2]]

## Cortes e inconsistencias del log

Cada punto de corte tiene sus propios COMMIT disponibles. Se clasifican las transacciones sin incorporar información que aparece después de ese punto.

[[TP8 - Cortes del log del ejercicio 3 y discrepancias de la fuente|Cortes del log del ejercicio 3 y discrepancias de la fuente]]

## Caso breve para practicar el método

El ejemplo independiente permite comprobar cómo cambia la recuperación al conservar o perder un COMMIT, sin depender de los datos inconsistentes del log impreso.

[[TP8 - Ejemplo breve de recuperación antes y después de COMMIT|Ejemplo breve de recuperación antes y después de COMMIT]]

## Tratar inconsistencias de la fuente

Si el valor anterior no se explica por las escrituras previas mostradas, se conserva la fuente y se declara la hipótesis, en lugar de corregirlo silenciosamente.

[[TP8 - Cortes del log del ejercicio 3 y discrepancias de la fuente|Cortes del log del ejercicio 3 y discrepancias de la fuente]]

## Fuente de estudio

Enunciado del TP8 en la carpeta `../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
