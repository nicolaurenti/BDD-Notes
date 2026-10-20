---
tags:
  - tp3
---

# Acciones referenciales


Se declaran en la tabla que contiene la FK, pero se activan al modificar o borrar la fila **referenciada**.

| Acción | Al borrar la fila referenciada | Al modificar su clave |
|---|---|---|
| `RESTRICT` | Rechaza si hay referencias | Rechaza si hay referencias afectadas |
| `CASCADE` | Borra las filas que la referencian | Propaga la nueva clave a las FK |
| `SET NULL` | Pone la FK afectada en nulo | Pone la FK afectada en nulo |
| `SET DEFAULT` | Asigna el valor por defecto | Asigna el valor por defecto |
| `NO ACTION` | Exige que la FK quede satisfecha al comprobarla | Lo mismo |

`NO ACTION` puede permitir comprobación diferida si la restricción se declaró diferible. `RESTRICT` no difiere esa acción. No asumir que son siempre intercambiables.

- `SET NULL` puede fallar si una columna es `NOT NULL` o parte de una PK.
- `SET DEFAULT` también debe dejar una referencia válida.
- Una cascada puede alcanzar otras tablas; si otra restricción impide el resultado, falla toda la sentencia.
- Cambiar directamente una FK en la tabla hija exige que el nuevo padre exista; `ON UPDATE CASCADE` no crea padres.
- Borrar una fila hija no borra automáticamente su padre.

Base técnica: [restricciones y acciones referenciales en PostgreSQL](https://www.postgresql.org/docs/current/ddl-constraints.html#DDL-CONSTRAINTS-FK).
