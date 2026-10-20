---
tags:
  - tp3
---

# MATCH SIMPLE, FULL y PARTIAL

Controlan los nulos de una **FK compuesta**.

| Modalidad | Regla |
|---|---|
| **SIMPLE** | Si hay algún NULL, no busca coincidencia. Sin nulos, exige la pareja exacta. |
| **FULL** | Todos NULL o pareja exacta sin nulos. Rechaza nulos parciales. |
| **PARTIAL** | Los valores informados deben coincidir en una misma fila padre. Todos NULL se admiten. |

## Cómo se declara

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

## Qué acepta cada uno

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
