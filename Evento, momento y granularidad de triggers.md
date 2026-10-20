---
tags:
  - tp4
---

# Evento, momento y granularidad de triggers


| Dimensión | Opciones | Pregunta |
|---|---|---|
| Evento | INSERT, UPDATE, DELETE | ¿Qué modificación activa el código? |
| Momento | BEFORE, AFTER, INSTEAD OF | ¿Modificar la fila, actuar después o reemplazar una operación sobre vista? |
| Granularidad | FOR EACH ROW / FOR EACH STATEMENT | ¿Una ejecución por fila o por sentencia? |

Para una sentencia que modifica 8 filas, un trigger de fila se ejecuta 8 veces y uno de sentencia una vez. Si no afecta filas, el de fila no se ejecuta; el de sentencia sí se dispara.

`BEFORE ROW` permite ajustar `NEW`. `AFTER` sirve para observar cambios realizados, aunque aún no estén confirmados por COMMIT. `INSTEAD OF ROW` se usa sobre vistas; ver TP5 - Vistas. [Sintaxis PostgreSQL](https://www.postgresql.org/docs/current/sql-createtrigger.html).
