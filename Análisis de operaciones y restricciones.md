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
