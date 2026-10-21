---
tags:
  - tp4
---

# TP4 - Auditoría STATEMENT y conteo de filas afectadas

Desarrollo del conteo global, las filas inexistentes y las bajas que se propagan por referencias.

## Qué cambia con STATEMENT

Si una sentencia borra tres entregas, ROW registra tres filas para ENTREGA. STATEMENT ejecuta el cuerpo una vez para esa sentencia, incluso si no borró nada. Cambiar únicamente FOR EACH ROW por FOR EACH STATEMENT **no convierte el 1 del ejemplo en 3**.

COUNT(*) de ENTREGA después de borrar cuenta las filas que quedan, no las eliminadas. OLD y NEW tampoco representan todas las filas en STATEMENT. Para guardar una cantidad correcta hace falta diseñar un mecanismo adicional de conteo: preparar el conteo al comenzar la sentencia, acumular una unidad por cada fila afectada y emitir un único registro al final. Ese diseño debe distinguir tabla y sentencia; un contador compartido sin distinguirlas puede mezclar operaciones. El cambio de granularidad, por sí solo, solo resuelve la frecuencia de ejecución.

En el DELETE del ejercicio 2, el número concreto requiere conocer las entregas seleccionadas y la acción de la FK de sus renglones. Si hay baja en cascada, también se auditan las bajas efectivamente producidas en RENGLON_ENTREGA. No asumir una única fila de auditoría total para ambas tablas ni una cantidad fija sin revisar las referencias.

## Fuente de estudio

Enunciado del TP4 en la carpeta `../Practicos` y filminas de cátedra `BDI_04 (SQL Procedural).pdf` (carpeta de cursada 2023).
