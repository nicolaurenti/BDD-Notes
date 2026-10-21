---
tags:
  - tp3
---

# Análisis de operaciones y restricciones


| Ejercicio | Idea que hay que dominar |
|---|---|
| 1 | Incorporar RIR, simular cascadas y matching. Cada operación parte de la instancia original. |
| 2 | Propagar efectos acumulados: una sentencia aceptada modifica el punto de partida de la siguiente. |
| 3 | Dominio, fechas, implicaciones y máximos de palabras por artículo. |
| 4 | Límite de productos, prefijo literal, nulidad conjunta y localidad proveedor–sucursal. |
| 5 | Edad, comparación con coordinador/tarea e historial anual de cambios de institución. |

Para cada operación: identificar filas afectadas, seguir todas las FK, comprobar nulidad y unicidad y justificar aceptación o rechazo. Si una sentencia falla, no conservar efectos parciales de esa sentencia. En una transacción PostgreSQL, un error requiere resolver el estado abortado antes de seguir; las simulaciones en papel suelen tratar cada sentencia por separado.

Reglas delicadas del ejercicio 5:

- Una edad máxima cambia con el paso del tiempo aunque no haya escrituras: un control solo al insertar no mantiene la regla para siempre.
- Si cambian horas o tarea de un coordinador, revisar también sus coordinados.
- Cambiar límites de una tarea puede invalidar voluntarios existentes.
- Para «cambios de institución por año», definir qué evento histórico representa un cambio; no contar automáticamente toda tarea histórica como cambio de institución.

## Una forma de presentar la justificación

Para cada sentencia, escribir: filas seleccionadas, valores propuestos, reglas afectadas y estado resultante. Por ejemplo, con un padre P1 y dos hijas que lo referencian mediante RESTRICT, una baja de P1 selecciona una fila pero se rechaza porque existen referencias. Una baja de P9 inexistente selecciona cero filas: no hay padre ni referencias que eliminar.

En el ejercicio 1 se restaura mentalmente la instancia original antes de cada inciso. En el 2 se conserva el resultado de cada sentencia aceptada. Si una sentencia rechazada proponía cambiar un identificador, el inciso siguiente sigue encontrando el identificador anterior, no el propuesto.

Para clasificar reglas, comenzar escribiendo la consulta que permitiría detectar una infracción. Si se necesitan dos columnas de PRODUCTO, el alcance es tupla; si se cuentan filas de PROVEE, es tabla; si se comparan localidades de PROVEEDOR y SUCURSAL a través de PROVEE, es general. Luego elegir el recurso SQL-1999 y responder por separado si PostgreSQL admite esa declaración.

## Reglas sobre el historial

«No más de tres cambios de institución en un año» cuenta cambios, no instituciones distintas ni tareas distintas. Una persona puede cambiar cuatro veces alternando entre dos instituciones. Si HISTORICO no distingue cuándo cambió de institución frente a cuándo cambió de tarea, hay que explicar qué información falta; no inventar que cualquier fila histórica representa un cambio de institución.

## Fuente de estudio

Enunciado del TP3 en la carpeta `../Practicos` y filminas de cátedra `BDI_03.P1 (Restricciones-1).pdf; BDI_03.P1 (Restricciones-2).pdf` (carpeta de cursada 2023).
