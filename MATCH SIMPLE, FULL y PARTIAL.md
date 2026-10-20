---
tags:
  - tp3
---

# Integridad referencial y matching

## Acciones referenciales

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

## MATCH SIMPLE, FULL y PARTIAL

Controlan los nulos de una **FK compuesta**.

| Modalidad | Regla |
|---|---|
| **SIMPLE** | Si hay algún NULL, no busca coincidencia. Sin nulos, exige la pareja exacta. |
| **FULL** | Todos NULL o pareja exacta sin nulos. Rechaza nulos parciales. |
| **PARTIAL** | Los valores informados deben coincidir en una misma fila padre. Todos NULL se admiten. |

### Cómo se declara

```sql
CREATE TABLE cliente (
    zona CHAR(1),
    nro_cliente INTEGER,
    PRIMARY KEY (zona, nro_cliente)
);

INSERT INTO cliente VALUES ('A', 1);

CREATE TABLE pedido (
    id_pedido INTEGER PRIMARY KEY,
    zona CHAR(1),
    nro_cliente INTEGER,
    FOREIGN KEY (zona, nro_cliente)
        REFERENCES cliente (zona, nro_cliente)
        MATCH SIMPLE
);
```

Para elegir otra modalidad, reemplazar `MATCH SIMPLE` por `MATCH FULL` o, en SQL estándar, `MATCH PARTIAL`. Las columnas de la FK admiten nulos en este ejemplo.

### Qué acepta cada uno

La única fila de `cliente` es **`('A', 1)`**:

| FK que se intenta insertar en pedido | SIMPLE | FULL | PARTIAL |
|---|:---:|:---:|:---:|
| `('A', 1)` | Sí | Sí | Sí |
| `('A', 9)` | No | No | No |
| `('A', NULL)` | Sí | No | Sí: coincide A |
| `(NULL, 1)` | Sí | No | Sí: coincide 1 |
| `('Z', NULL)` | Sí | No | No: no existe zona Z |
| `(NULL, NULL)` | Sí | Sí | Sí |

Ejemplo concreto:

```sql
INSERT INTO pedido VALUES (10, 'Z', NULL);
-- SIMPLE: acepta por tener un NULL.
-- FULL: rechaza porque solo una parte es NULL.
-- PARTIAL: rechazaría porque ninguna fila padre tiene zona Z.
```

**PostgreSQL:** SIMPLE es el valor por defecto; FULL está soportado; PARTIAL no está implementado. [Referencia](https://www.postgresql.org/docs/current/sql-createtable.html).
