---
tags:
  - tp4
---

# OLD, NEW y retornos de triggers


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
