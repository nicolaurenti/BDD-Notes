---
tags:
  - tp3
---

# MATCH SIMPLE, FULL y PARTIAL


Supongamos una FK `(zona, nro_cliente)` que admite nulos y una fila padre `('A', 1)`.

| FK de la fila hija | `SIMPLE` | `FULL` | `PARTIAL` del estándar |
|---|---|---|---|
| `('A', 1)` | Acepta | Acepta | Acepta |
| `('Z', 9)` sin padre | Rechaza | Rechaza | Rechaza |
| `('A', NULL)` | Acepta | Rechaza | Acepta si existe algún padre con zona A |
| `('Z', NULL)` sin padre en zona Z | Acepta | Rechaza | Rechaza |
| `(NULL, NULL)` | Acepta | Acepta | Acepta |

- **SIMPLE:** si cualquier componente es nulo, no exige buscar una coincidencia para esa FK.
- **FULL:** admite todos nulos o todos informados con una coincidencia exacta; rechaza la mezcla.
- **PARTIAL:** los componentes informados deben coincidir en una misma fila padre; todos nulos se admiten.

PostgreSQL implementa `SIMPLE` y `FULL`; no implementa `PARTIAL`. Si no se indica matching, se usa `SIMPLE`. [Referencia de CREATE TABLE](https://www.postgresql.org/docs/current/sql-createtable.html).
