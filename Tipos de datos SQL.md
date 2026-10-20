---
tags:
  - tp1
---

# Tipos de datos SQL


| Tipo | Uso |
|---|---|
| `INTEGER` | Números enteros. |
| `NUMERIC(p,s)` / `DECIMAL(p,s)` | Números exactos: `p` indica la cantidad total de dígitos y `s` la cantidad de decimales. |
| `CHAR(n)` | Texto de longitud fija. |
| `VARCHAR(n)` | Texto de longitud variable, hasta `n` caracteres. |
| `DATE` | Fechas. |

**Ejemplos:**

- `DECIMAL(5,0)`: hasta cinco dígitos, sin decimales.
- `DECIMAL(7,2)`: hasta siete dígitos en total, de los cuales dos son decimales.
- `CHAR(2)`: código de idioma de longitud fija.
- `VARCHAR(120)`: título de hasta 120 caracteres.
