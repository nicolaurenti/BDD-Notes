---
tags:
  - tp4
---

# Fundamentos y ejecución de triggers

Un trigger asocia un evento de modificación con código que el motor ejecuta automáticamente. En este práctico se usa para expresar controles que no pudieron implementarse declarativamente en el TP3 y para mantener auditorías e información derivada. Una regla declarativa expresa el estado permitido; el trigger debe decidir **cuándo comprobarlo y qué hacer**.

## Trigger, función y stored procedure

La función contiene el cuerpo y el trigger declara tabla, eventos, momento y granularidad. Se usa RETURNS trigger y la cabecera EXECUTE PROCEDURE del enunciado. Un procedimiento común se invoca explícitamente con CALL.

[[TP4 - Anatomía de una función y su trigger|Anatomía de una función y su trigger]]

## Decisiones de ejecución

Evento, momento y granularidad son decisiones independientes: qué operación dispara, cuándo actúa el cuerpo y si se ejecuta por fila o por sentencia.

[[TP4 - Evento, momento y granularidad del trigger|Evento, momento y granularidad del trigger]]

## Filas y retornos

OLD representa la fila anterior y NEW la propuesta, según el evento. El retorno permite continuar, omitir una fila o indicar una operación atendida; una infracción se rechaza con RAISE EXCEPTION.

[[TP4 - OLD, NEW, retornos y variables del trigger|OLD, NEW, retornos y variables del trigger]]

## Caso trabajado de granularidad

La simulación exige ubicar cada ejecución antes de la fila o de la sentencia y actualizar el estado entre pasos. El nombre de la función no reemplaza la lectura de su operación.

[[TP4 - Simulación de sueldos con ROW y STATEMENT|Simulación de sueldos con ROW y STATEMENT]]

## Fuente de estudio

Enunciado del TP4 en la carpeta `../Practicos` y filminas de cátedra `BDI_04 (SQL Procedural).pdf` (carpeta de cursada 2023).
