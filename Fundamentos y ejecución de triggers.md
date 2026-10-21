---
tags:
  - tp4
---

# Fundamentos y ejecución de triggers

## Triggers, funciones y procedimientos

| Recurso | Cómo se ejecuta | Uso |
|---|---|---|
| Restricción declarativa | La verifica el motor | PK, FK, nulidad y reglas expresables con CHECK |
| Trigger | Automáticamente ante un evento | Validación, auditoría, historial y mantenimiento de derivados |
| Función | Mediante una expresión o desde otro código | Calcular o encapsular lógica; puede modificar datos según su definición |
| Procedimiento | Mediante `CALL` | Carga inicial o proceso invocado explícitamente |

Un procedimiento no garantiza mantenimiento permanente si se puede actualizar la tabla sin llamarlo. Un trigger tampoco reconstruye automáticamente el pasado.

## Evento, momento y granularidad de triggers

| Dimensión | Opciones | Pregunta |
|---|---|---|
| Evento | INSERT, UPDATE, DELETE | ¿Qué modificación activa el código? |
| Momento | BEFORE, AFTER, INSTEAD OF | ¿Modificar la fila, actuar después o reemplazar una operación sobre vista? |
| Granularidad | FOR EACH ROW / FOR EACH STATEMENT | ¿Una ejecución por fila o por sentencia? |

Para una sentencia que modifica 8 filas, un trigger de fila se ejecuta 8 veces y uno de sentencia una vez. Si no afecta filas, el de fila no se ejecuta; el de sentencia sí se dispara.

`BEFORE ROW` permite ajustar `NEW`. `AFTER` sirve para observar cambios realizados, aunque aún no estén confirmados por COMMIT. `INSTEAD OF ROW` se usa sobre vistas; ver TP5 - Vistas. [Sintaxis PostgreSQL](https://www.postgresql.org/docs/current/sql-createtrigger.html).

## OLD, NEW y retornos de triggers

| Evento de fila | OLD | NEW |
|---|---|---|
| INSERT | No disponible | Fila propuesta |
| UPDATE | Fila anterior | Fila propuesta |
| DELETE | Fila anterior | No disponible |

- En `BEFORE INSERT/UPDATE`, devolver `NEW` para continuar con esa fila.
- En `BEFORE DELETE`, devolver `OLD` para permitir el borrado.
- `RETURN NULL` en un trigger `BEFORE ROW` omite la operación para esa fila; no lanza un error.
- En `AFTER` y triggers de sentencia, el retorno se ignora: es habitual `RETURN NULL`.
- Para rechazar por una infracción, usar `RAISE EXCEPTION`.

Variables útiles: `TG_OP`, `TG_TABLE_NAME`, `TG_WHEN` y `TG_LEVEL`. En un trigger de sentencia no existe una única fila `NEW/OLD`. [Funciones de trigger](https://www.postgresql.org/docs/current/plpgsql-trigger.html).

## Simulación de triggers ROW y STATEMENT

El código del ejercicio resta un 5 % del sueldo mínimo; el nombre «autoincremento» no cambia lo que hace. Antes de interpretarlo, corregir terminadores y retorno según la granularidad.

Suponiendo el orden de inserción 700, 300, 700 y sueldos NUMERIC, el empleado inicial cobra 500:

| BEFORE ROW | Mínimo antes de cada fila | Descuento a filas ya presentes |
|---|---|---|
| Antes de insertar 700 | 500 | 25 |
| Antes de insertar 300 | 475 | 23.75 |
| Antes de insertar el último 700 | 300 | 15 |

Resultado por id: inicial `436.25`; id 2 `661.25`; id 3 `285`; id 4 `700`. En `BEFORE STATEMENT`, se descuenta una vez al inicial: `475`; los tres nuevos conservan `700, 300, 700`.

No asumir orden de procesamiento de un `INSERT ... SELECT` sin orden explícito. Si cambia el orden o el tipo numérico, revisar el resultado. Para simular: ejecutar mentalmente el trigger antes de cada inserción, no después de todas.
