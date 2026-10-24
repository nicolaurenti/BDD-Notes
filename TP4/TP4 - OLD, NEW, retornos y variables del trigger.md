---
tags:
  - tp4
---

# TP4 - OLD, NEW, retornos y variables del trigger

Tabla de disponibilidad por evento, retornos y variables de contexto.

## OLD, NEW y RETURN

| Evento ROW | OLD | NEW |
|---|---|---|
| INSERT | No hay fila anterior | Fila a incorporar |
| UPDATE | Valores anteriores | Valores propuestos |
| DELETE | Fila a eliminar | No hay fila nueva |

En BEFORE INSERT/UPDATE, RETURN NEW permite continuar usando la fila propuesta, incluidas las modificaciones que hizo la función. En BEFORE DELETE se devuelve OLD para permitir la baja. RETURN NULL en BEFORE ROW omite esa fila: **no equivale a rechazar la sentencia con un error**. Para una infracción se usa RAISE EXCEPTION.

En AFTER y en triggers de sentencia el valor devuelto se ignora; se puede escribir RETURN NULL. Un trigger STATEMENT no tiene una única fila OLD o NEW que represente todo el conjunto.

`TG_OP` informa la operación; `TG_TABLE_NAME`, la tabla; `TG_WHEN`, el momento; y `TG_LEVEL`, la granularidad. Sirven para compartir un cuerpo entre varios eventos o identificar qué se está auditando.

## Fuente de estudio

Enunciado del TP4 en la carpeta `../../Practicos` y filminas de cátedra `BDI_04 (SQL Procedural).pdf` (carpeta de cursada 2023).
